---
layout: post
title:  "Quick Run"
date:   2020-05-02 21:28:21 -0400
category: [writeups]
challenge-source: [VirSecCon2020]
challenge-category: ['Scripting']
tags: [VirSecCon, ctf, scripting]
author: shinris3n
---

This challenge contained a zip file with 30 QR codes inside.

![Partial QR Codes](/assets/writeups/VirSecCon2020/Quick_Run/05-02-2020-225222.png)

There is a tool called zbarimg that can process QR codes via command line.

{% highlight shell_session %}
$ sudo apt-get install zbar-tools
{% endhighlight %}

&nbsp;&nbsp;&nbsp; ![zbarimg syntax](/assets/writeups/VirSecCon2020/Quick_Run/05-02-2020-231557.png)

Just checking the first file:

{% highlight shell_session %}
$ zbarimg 0.png
{% endhighlight %}

&nbsp;&nbsp;&nbsp; ![zbarimg output](/assets/writeups/VirSecCon2020/Quick_Run/05-02-2020-232327.png)

We can guess that if this is a character in flag, it is likely an ASCII encoded character.  We can check a table:  

![ASCII Table](/assets/writeups/VirSecCon2020/Quick_Run/ASCII.png)

We know flags in this competition begin with an L, so this is probably decimal encoded rather than hex encoded.

One option is just to use zbarimg to go through all the files in the directory and match the characters to the table, but - in the spirit of the category - to completely automate the process via a python script:

{% highlight python %}
import os
characters = ''
a = 0
while (a <= 30):
        qrcode = str(a) + ".png"
        #print (qrcode)
        command_to_run = "zbarimg " + qrcode + " --quiet --raw"
        character = os.popen(command_to_run).read()
        character = character [8:-1]
        character = hex(int(character))
        character = bytes.fromhex(character[2:]).decode('ascii')
        characters = characters + character
        a = a + 1
print (characters)
{% endhighlight %}

&nbsp;&nbsp;&nbsp; ![Python Script Output](/assets/writeups/VirSecCon2020/Quick_Run/05-02-2020-235110.png)