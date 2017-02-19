---
layout: post
title:  URIs don't allow UTF-8
date:   2017-02-19 17:34:00 +0000
source: https://tools.ietf.org/html/rfc3987
categories: http
---

As developers, we increasingly live in a [Unicode][unicode] world. Our code is
written in Unicode, we output in Unicode and we - mostly - expect input to be
in Unicode.

An interesting bug however alerted to me something that had completely missed
me: A HTTP resource identifier with Unicode characters is not a [URI][uri].

For example there is an article on [Al Jazeera][al-jazeera] about Lionel Messi
with an identifier of:
[`http://sport.aljazeera.net/ligabbva/2017/2/19/هل-يرفض-ميسي-124-مليون-دولار`][messi].
You can click this and visit the page, and this address will show up in your
browser bar - but this is not a URI. This is actually an [IRI][iri] -
International Resource Identifier - and it differs from a URI by the presence
of Unicode characters.

An IRI is not something a browser requests. Instead a browser transparently
converts the IRI into a URI by [percent encoding][percent-encoding] the Unicode
characters.

So when you request `http://sport.aljazeera.net/ligabbva/2017/2/19/هل-يرفض-ميسي-124-مليون-دولار`
your browser is actually requesting `http://sport.aljazeera.net/ligabbva/2017/2/19/%D9%87%D9%84-%D9%8A%D8%B1%D9%81%D8%B6-%D9%85%D9%8A%D8%B3%D9%8A-124-%D9%85%D9%84%D9%8A%D9%88%D9%86-%D8%AF%D9%88%D9%84%D8%A7%D8%B1` -
not so friendly to humans.

Modern web browsers perform this conversion on your behalf, but this not the
case with a tool like [curl][curl] where you are expected to request a URI
rather than an IRI.

How a server will respond to the presence of Unicode in a URI is inconsistent.
Al Jazeera responds with a 400, as does Github. However Google, Twitter and
Facebook accept them.

```
➜  ~ curl -Is http://sport.aljazeera.net/ligabbva/2017/2/19/%D9%87%D9%84-%D9%8A%D8%B1%D9%81%D8%B6-%D9%85%D9%8A%D8%B3%D9%8A-124-%D9%85%D9%84%D9%8A%D9%88%D9%86-%D8%AF%D9%88%D9%84%D8%A7%D8%B1 | grep HTTP
HTTP/1.1 200 OK
➜  ~ curl -Is http://sport.aljazeera.net/ligabbva/2017/2/19/هل-يرفض-ميسي-124-مليون-دولار | grep HTTP
HTTP/1.1 400 Bad Request
➜  ~ curl -Is https://www.github.com/\?\=🐧 | grep HTTP
HTTP/1.0 400 Bad Request
➜  ~ curl -Is https://www.google.co.uk/\?\=🐧 | grep HTTP
HTTP/1.1 200 OK
➜  ~ curl -Is https://www.facebook.com/\?\=🐧 | grep HTTP
HTTP/1.1 200 OK
➜  ~ curl -Is https://twitter.com/\?\=🐧 | grep HTTP/1.1
HTTP/1.1 200 OK
```

Although the IRI standard has now existed for 12 years it can be difficult to
support the conversion from IRI to URI in programming languages. For example
Ruby provides support through it's [`URI::Escape.escape`][ruby-uri-escape]
method, however for JavaScript you're probably going to need a
[package][node-iri-package].

👻

[al-jazeera]: http://www.aljazeera.net
[curl]: https://curl.haxx.se/
[iri]: https://tools.ietf.org/html/rfc3987
[messi]: http://sport.aljazeera.net/ligabbva/2017/2/19/هل-يرفض-ميسي-124-مليون-دولار
[node-iri-package]: https://www.npmjs.com/package/iri
[percent-encoding]: https://en.wikipedia.org/wiki/Percent-encoding
[ruby-uri-escape]: http://ruby-doc.org/stdlib-1.9.3/libdoc/uri/rdoc/URI/Escape.html#method-i-escape
[unicode]: https://en.wikipedia.org/wiki/Unicode
[uri]: https://tools.ietf.org/html/rfc3986
