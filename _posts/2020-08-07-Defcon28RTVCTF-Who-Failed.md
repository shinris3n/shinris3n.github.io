---
layout: post
title:  "Who Failed"
date:   2020-08-07 22:19:10 -0400
category: [writeups]
challenge-source: [Defcon 28 - Red Team Village CTF]
challenge-category: ['logs']
tags: [Defcon28-RTVCTF]
author: shinris3n
---

Logs - Who Failed

The challenge seems easy enough:

*How many different IP addresses were banned?*

However, we don't want to have to go through and count these manually. Looking at the file, we can see how a ban is formatted:

![47da493d350f31e48f33b794498088a1.png](/assets/writeups/DefconRTVCTF/784a74acfcc049828cc0ca2c7cf2c6ae.png)

So we can combine grep and sort as such:

{% highlight bash %}
cat fail2ban.log | grep -oe "Ban [0-9.]\{11,19\}" | sort --unique
{% endhighlight %}

This gives us unique instances of any strings that look like "Ban X.X.X.X" (11 characters) through "Ban XXX.XXX.XXX.XXX" (19 characters), where X is a number between 0 and 9 or a period.

![91d5c4fd898f54905c70c7099c98f563.png](/assets/writeups/DefconRTVCTF/f4b4c861b46a4722ab99e29531654ddd.png)

With a bigger list we would also probably want to leverage the wc command to count the number of lines.  So a complete, single line solution for this challenge could be:

{% highlight bash %}
cat fail2ban.log | grep -oe "Ban [0-9.]\{11,19\}" | sort --unique | wc -l
{% endhighlight %}

![05fbc0787b714dbeaa71cacdc5504dd7.png](/assets/writeups/DefconRTVCTF/a4d17f6322cf4b49bbf6f849fb3b4000.png)

