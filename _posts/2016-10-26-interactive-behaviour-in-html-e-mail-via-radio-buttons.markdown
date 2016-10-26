---
layout: post
title:  Interactive Behaviour in HTML e-mail via Radio Buttons
date:   2016-10-26 20:06:00 +0100
source: http://communicatoremail.com/MNE4ftRQ_R6lma4Bv3Yl_F1INQC5MA7pqvn6lscejgK/WebView.aspx
categories: rails
---
Constraints breed innovation? This seems to be frequently the case in the world
of HTML e-mail. Despite a [rocky landscape of restrictions][cm-css] people
have acheived some [incredible][gmail-responsive] [things][video-email]. I was
surprised however today to get an [interactive email][dominos] from
[Dominos](https://www.dominos.co.uk/) where you can click to see
different images in a tab like interface.

Digging into view source I could see the usage was achieved using a `<label>` element
with a `<input type="radio">` inside it. Once the input is checked (via clicking the label)
the [adjacent CSS selector][adjacent](`+`) is used to change the styling of the image.

This technique isn't actually new, radio buttons have been a commonly used trick
for behaviour without Javascript - known as "[checkbox hack][checkbox-hack]".
Famously used for a [HTML + CSS version][css-minesweeper] of Minesweeper.

Since forms are available in e-mails and many clients support modern CSS it makes
sense this works, however this is the first time I've seen it. I found a couple
of [other][hamburger-email] [examples][click-bait] searching the web.

Nice work Dominos 👏

[cm-css]: https://www.campaignmonitor.com/css/
[gmail-responsive]: https://julie.io/writing/gmail-first-strategy-for-responsive-emails/
[video-email]: https://litmus.com/blog/how-to-code-html5-video-background-in-email
[dominos]: http://communicatoremail.com/MNE4ftRQ_R6lma4Bv3Yl_F1INQC5MA7pqvn6lscejgK/WebView.aspx
[adjacent]: https://developer.mozilla.org/en-US/docs/Web/CSS/Adjacent_sibling_selectors
[checkbox-hack]: https://css-tricks.com/the-checkbox-hack/
[css-minesweeper]: http://jsdo.it/No_1026/urFs
[hamburger-email]: https://litmus.com/community/discussions/999-hamburger-in-email
[click-bait]: https://www.campaignmonitor.com/blog/email-marketing/2016/09/7-email-hacks-every-developer-should-know/
