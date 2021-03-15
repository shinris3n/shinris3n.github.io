---
layout: post
title:  "Pollex"
date:   2021-03-15 00:08:00 -0400
category: [writeups]
challenge-source: [NahamCon CTF 2021]
challenge-category: ['warmups']
tags: [NahamConCTF2021]
author: shinris3n
---
<h1></h1>
<h3><font color=FF3030>Challenge Text</font></h3>

```
Author: @JohnHammond#6971

👍

Download the file below.

Some people seem to have trouble reading this, understandably so. Sorry. The flag ends in these characters: 8fe36bc00}
``` 

<h1></h1>
<h3><font color=FF3030>Solution</font></h3>

- Opening the image file provided, nothing seems out of place.  However, if we look closely at the thumbnail theres a flag hidden there:

![593b7c4d875d4da3b29be9e116e24d0e.png](/assets/writeups/NahamConCTF2021/593b7c4d875d4da3b29be9e116e24d0e.png)

- Let's not fall for John Hammond's pro trolling attempts by using our biggest monitor and accessibility settings to try to see the image as it appears in our folder or taking screenshots and trying to zoom in via some image editor.  (Yes, I may have done this at first...)  

- Instead, we can easily see the flag clearly using exiftool to extract the thumbnail image:

{% highlight bash%}
exittool -b -Thumbnailimage pollex > ./pollex_thumbnail
{% endhighlight %}

![c66d60c7816a44508a440b49dde15d7f.png](/assets/writeups/NahamConCTF2021/c66d60c7816a44508a440b49dde15d7f.png)