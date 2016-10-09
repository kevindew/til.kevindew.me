---
layout: post
title:  PostgreSQL Transaction Isolation Levels
date:   2016-10-09 17:36:00 +0100
source: https://www.postgresql.org/docs/9.5/static/transaction-iso.html
categories: postgresql
---
In [SQL Standard][sql-92-isolation] there are 4 levels of transaction isolation
defined: `READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ` and `SERIALIZABLE`.

Of these PostgreSQL supports `READ COMMITTED`, `REPEATABLE READ` and
`SERIALIZABLE`. Where `READ COMMITTED` is the default.

### Examples of each isolation level

#### DB Setup

```
transactions# CREATE TABLE albums (id serial PRIMARY KEY, title varchar NOT NULL, rating DECIMAL(2) );
CREATE TABLE
transactions# INSERT INTO albums (title, rating) values ('Rust in Peace', 10), ('Countdown to Extinction', 9);
INSERT 0 2
```

#### READ COMMITTED

The results of queries within this transaction will reflect committed
transactions that have happened outside this transaction (thus within this
transaction the same query could yield different results) and queries that are
executed within this transaction.

```
transactions# BEGIN;
BEGIN
transactions# -- has no effect as this is the default isolation level
transactions# SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET
transactions*# SELECT COUNT(*) FROM ALBUMS;
┌───────┐
│ count │
├───────┤
│     2 │
└───────┘
(1 row)

other-session# INSERT INTO albums (title, rating) VALUES ('Peace Sells... but Who''s Buying?', 8);
INSERT 0 1
transactions*# SELECT COUNT(*) FROM albums;
┌───────┐
│ count │
├───────┤
│     3 │
└───────┘
(1 row)

transactions*# INSERT INTO albums (title, rating) VALUES ('The World Needs a Hero', 1);
INSERT 0 1
transactions*# SELECT COUNT(*) FROM albums;
┌───────┐
│ count │
├───────┤
│     4 │
└───────┘
(1 row)

transactions*# ROLLBACK;
ROLLBACK
other-session# DELETE FROM albums WHERE title = 'The World Needs a Hero';
DELETE 1
```

#### REPEATABLE READ

This isolation level will not see changes that are committed in other
transactions while this transaction is open. Transactions on the dataset will
be allowed to be committed while this transaction is running.

```
transactions# BEGIN;
BEGIN
transactions# SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET
transactions*# SELECT COUNT(*) FROM ALBUMS;
┌───────┐
│ count │
├───────┤
│     2 │
└───────┘
(1 row)

other-session# INSERT INTO albums (title, rating) VALUES ('Peace Sells... but Who''s Buying?', 8);
INSERT 0 1
transactions*# SELECT COUNT(*) FROM albums;
┌───────┐
│ count │
├───────┤
│     2 │
└───────┘
(1 row)

transactions*# INSERT INTO albums (title, rating) VALUES ('The World Needs a Hero', 1);
INSERT 0 1
transactions*# SELECT COUNT(*) FROM albums;
┌───────┐
│ count │
├───────┤
│     3 │
└───────┘
(1 row)

transactions*# ROLLBACK;
ROLLBACK
other-session# DELETE FROM albums WHERE title = 'The World Needs a Hero';
DELETE 1
```

#### SERIALIZABLE

`SERIALIZABLE` is very similar to `REPEATABLE READ` - They were actually
synonymous until PostgreSQL 9.1. However the key difference is that it treats
transactions as being executed in series and will error on situations where
the ordering of concurrent transactions presents different results.

Consider this pair of `REPEATABLE READ` transactions:

```
transactions# BEGIN;
BEGIN
transactions*# SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET
transactions*# INSERT INTO albums (title, rating) VALUES ('Peace Sells... but Who''s Buying?', 8);
INSERT 0 1
transactions*# INSERT INTO albums (title, rating) VALUES ('Greatest Hits', (SELECT AVG(rating) FROM albums));
INSERT 0 1
other-session# BEGIN;
BEGIN
other-session*# SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET
other-session*# INSERT INTO albums (title, rating) VALUES ('The World Needs a Hero', 1);
INSERT 0 1
other-session*# INSERT INTO albums (title, rating) VALUES ('Greatest Hits', (SELECT AVG(rating) FROM albums));
INSERT 0 1
other-session*# commit;
COMMIT
transactions*# commit;
COMMIT
transactions# SELECT * FROM albums;
┌────┬──────────────────────────────────┬────────┐
│ id │              title               │ rating │
├────┼──────────────────────────────────┼────────┤
│  1 │ Rust in Peace                    │     10 │
│  2 │ Countdown to Extinction          │      9 │
│  3 │ Peace Sells... but Who's Buying? │      8 │
│  4 │ Greatest Hits                    │      9 │
│  5 │ The World Needs a Hero           │      1 │
│  6 │ Greatest Hits                    │      7 │
└────┴──────────────────────────────────┴────────┘
(6 rows)
```

💩👎

Whereas with `SERIALIZABLE`:

```
transactions# BEGIN;
BEGIN
transactions*# SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SET
transactions*# INSERT INTO albums (title, rating) VALUES ('Peace Sells... but Who''s Buying?', 8);
INSERT 0 1
transactions*# INSERT INTO albums (title, rating) VALUES ('Greatest Hits', (SELECT AVG(rating) FROM albums));
INSERT 0 1
other-session# BEGIN;
BEGIN
other-session*# SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SET
other-session*# INSERT INTO albums (title, rating) VALUES ('The World Needs a Hero', 1);
INSERT 0 1
other-session*# INSERT INTO albums (title, rating) VALUES ('Greatest Hits', (SELECT AVG(rating) FROM albums));
INSERT 0 1
transactions*# COMMIT;
COMMIT
other-session*# COMMIT;
ERROR:  40001: could not serialize access due to read/write dependencies among transactions
DETAIL:  Reason code: Canceled on identification as a pivot, during commit attempt.
HINT:  The transaction might succeed if retried.
LOCATION:  PreCommit_CheckForSerializationFailure, predicate.c:4654
```

#### READ UNCOMMITTED

This level is unsupported by PostgreSQL and using it is treated as `READ
COMMITTED`. It is provided to allow read access to transactions that haven't
committed. So if we use our imagination:

```
transactions# BEGIN;
BEGIN
transactions*# -- Let's pretend this works in PostgreSQL
transactions*# SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SET
transactions*# SELECT COUNT(*) FROM albums;
┌───────┐
│ count │
├───────┤
│     2 │
└───────┘
(1 row)

other-session# BEGIN;
BEGIN
other-session*# INSERT INTO albums (title, rating) VALUES ('Hidden Treasures', 6);
INSERT 0 1
transactions*# SELECT COUNT(*) FROM albums;
┌───────┐
│ count │
├───────┤
│     3 │
└───────┘
(1 row)

other-session*# ROLLBACK;
ROLLBACK
transactions*# SELECT COUNT(*) FROM albums;
┌───────┐
│ count │
├───────┤
│     2 │
└───────┘
(1 row)

transactions*# END;
COMMIT
```

But in reality:

```
transactions# BEGIN;
BEGIN
transactions*# -- Actually acts the same as READ COMMITTED
transactions*# SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SET
transactions*# SELECT COUNT(*) FROM albums;
┌───────┐
│ count │
├───────┤
│     2 │
└───────┘
(1 row)

other-session# BEGIN;
BEGIN
other-session*# INSERT INTO albums (title, rating) VALUES ('Hidden Treasures', 6);
INSERT 0 1
transactions*# SELECT COUNT(*) FROM albums;
┌───────┐
│ count │
├───────┤
│     3 │
└───────┘
(1 row)

other-session*# ROLLBACK;
ROLLBACK
transactions*# SELECT COUNT(*) FROM albums;
┌───────┐
│ count │
├───────┤
│     2 │
└───────┘
(1 row)

transactions*# END;
COMMIT
```

[sql-92-isolation]: http://www.ibm.com/developerworks/data/zones/informix/library/techarticle/db_isolevels.html
