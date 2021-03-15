---
layout: post
title:  "Parseltongue"
date:   2021-03-15 00:33:00 -0400
category: [writeups]
challenge-source: [NahamCon CTF 2021]
challenge-category: ['forensics']
tags: [NahamConCTF2021]
author: shinris3n
---
<h1></h1>
<h3><font color=FF3030>Challenge Text</font></h3>


```
Author: @JohnHammond#6971

Hisssss, can you ssssee ssssome sssssecretssss?

Download the file below.
``` 

<h1></h1>
<h3><font color=FF3030>Solution</font></h3>

- Running strings on the file we can see it has an embedded python file in it.  With the help of [uncompyle6](https://pypi.org/project/uncompyle6/){:target="_blank"} we're able to decompile the binary (after changing its extension to pyc):

{% highlight bash %}
uncompyle6 pt.pyc > pt.py
{% endhighlight %}

{% highlight python %}
# uncompyle6 version 3.7.4
# Python bytecode 3.8 (3413)
# Decompiled from: Python 3.8.6 (default, Sep 25 2020, 09:36:53) 
# [GCC 10.2.0]
# Embedded file name: ./parseltongue.py
# Compiled at: 2021-02-28 08:42:29
# Size of source mod 2**32: 1889 bytes
import Crypto.Util.number as l2b
import random
sszz = [
 'aposlogahs', 'apsle', 'Sine', 'aʃe', 'bei∫ed', 'tuif', 'Kura', 'Vera', 'pard', 'pardshesl', 'bo∫', 'Gara', 'vinth', 'Pelʃis', 'keilsing', 'khair', 'tikni', 'Bana', 'Slehara', 'koukh', 'kups', 'dai', 'Andi', 'dorʃe', 'doʃe', 'sloʃe', 'kaʃe', 'Sarna', 'Suu', 'giʃe', 'Gorna', 'ass-girou', 'dros', 'feslure', 'hasli', 'riʃan', 'fraeslis', 'vris', 'gatsi', 'runʃe', 'Tira', 'hishe', 'einʃe', 'hesleuf', 'Firna', 'Baʃ', 'ʃem', 'ai', 'ine', 'dinʃe', 'Negei', 'slanp', 'ʃena', 'sliʃe', 'dati', 'slifai', 'Kuine', 'Ha', 'nisl', 'ʃe', 'Sobne', 'bna', 'Sora', 'ovith', 'houk', 'parknent', 'fasar', 'nesha', 'praughs', 'Pura', 'ʃine', 'ʃane', 'gisan', 'rai∫e', 'kata', 'Ara', 'Nigi', 'akaʃe', 'rashe', 'slan', 'Derne', 'Tina', 'snart', 'gariʃe', 'kerashe', 'stabsle', 'Fasi', 'Peina', 'Tasi', 'Sekusi', 'Harne', 'kapi', 'Athne', 'vaʃe', 'asl', 'ʃik', 'agiro', 'vei', 'Asuna', 'Teʃ', 'Fiʃ', 'Doʃ', 'ʃira', 'Haʃ', 'Vuʃ', 'vindovth', 'Bira', 'Sa', 'Slu', 'ou', 'iangsteur']
zzss = b'\x07\x1c\x0e\x14\x17\n\x06\x03\x0cJ\x00@G\x0e\x017X\x0b\x04W\xf8\xb5\x03P\x06\x0f\x80\xea\x9b\x00\x05A\x16\\\x00.\x17\x0f'
s = False
z = True
ss = s & z
z = abs(ss) - abs(z)
zz = ss | z
z = zz - z - z
zz = z | z
z = zz << zz
s = ss >> ss
sz = s << z
zs = z << s
z = zs - sz
ss = str(z).replace(str(zs), str(ss).replace(str(ss), str(z).replace(str(z), '')))
sss = bytes(ss.join(sszz), 'utf-8')
zzz = bytes([_a ^ _b for _a, _b in zip(sss, zzss)])
ssszzz = bytes([_a ^ _b for _a, _b in zip(zzz, zzss)])
sss += b'S'
ssss = []
ss = sss[:len(sss) // 2]
zz = sss[len(sss) // 2:]
for s in range(len(ss)):
    ssss.append(ss[s] ^ zz[s])
else:
    if 5 == 1:
        print(' '.join([random.choice(sszz).upper() for _ in range(random.randrange(5, 10))]))
    else:
        print(' '.join([random.choice(sszz).upper() for _ in range(random.randrange(5, 10))]))
# okay decompiling pt.pyc
{% endhighlight %}

- There's a lot going on here and many similar variables to confuse things.  When we run the program we know we're only seeing output that takes a few random elements of the sszz array and capitalizes the characters in the string.  

- To that end, the easiest thing to do here first is just modify the code to print the output of the other variables - "zzz" and "ssszzz" stand out as suspicious because they are iterating over the "zzss" string of bytecode.  Just trying these first, the flag string turns out to be hiding in the "zzz" variable.

{% highlight python%}
print (zzz)
print (ssszzz)
{% endhighlight %}

{% highlight bash %}
$ python3 py.py                                                          
APSLE INE AGIRO HAƩ BANA GARA SINE SLANP
b'flag{eabd9a04bdd1ea626f2cfbb0ea5c5feb}'
b'aposlogahsapsleSinea\xca\x83ebei\xe2\x88\xabedtuifKur'
```
{% endhighlight %}