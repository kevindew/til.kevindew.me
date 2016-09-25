---
layout: post
title:  "Copy multiple files to a single destination"
date:   2016-09-25 18:26:47 +0100
source: http://askubuntu.com/questions/327282/copying-multiple-specific-files-from-one-folder-to-another
categories: unix
---
Using a [glob][glob] pattern can allow you to copy multiple files with distinct names
to the same destination.

{% highlight shell %}
cp {foo.txt,bar.txt} dest/
{% endhighlight %}

With [ZSH][zsh] you also get tab completion within the curly brackets 👍

[glob]: http://www.tldp.org/LDP/GNU-Linux-Tools-Summary/html/x11655.htm
[zsh]: http://www.zsh.org/
