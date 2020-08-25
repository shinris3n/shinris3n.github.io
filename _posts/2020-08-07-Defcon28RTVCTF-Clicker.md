---
layout: post
title:  "Clicker"
date:   2020-08-07 22:20:15 -0400
category: [writeups]
challenge-source: [Defcon 28 - Red Team Village CTF]
challenge-category: ['pwn']
tags: [Defcon28-RTVCTF]
author: shinris3n
---

Pwn - Clicker

*"All you have to do is score 10,000,000 to get the flag. Get Clicking!!!!!"*

First we download clicker and make it executable.

![b1d1a968f62062b7aa872004dfdcbc29.png](/assets/writeups/DefconRTVCTF/4538f00cb9654440b1906f891e1e2c14.png)

Well we can hit or hold enter to buy toilet paper and buy a multiplier once we have enough rolls.  This resets the number of rolls to 0, and then we can buy 2 per "click".  Yep, there's no way we're getting to 10 million like this.

So what happens when we save?

![f41d2846978714b65841a38cba031032.png](/assets/writeups/DefconRTVCTF/5f208191b1be4822a4504294fefa2fc3.png)

![e3e22423bbcf66573ee4db58a42bb5ea.png](/assets/writeups/DefconRTVCTF/09e4172a4c234ad493eea3d0f06e3b69.png)

Just text?  That seems easy enough to mess with!

![1f6af210cec12a7fb7dc6341c9442709.png](/assets/writeups/DefconRTVCTF/339b34060b02462ea7cbe6d7d6255135.png)

![f190dba4f16e3d85ac7e1ea7f4a3fcb4.png](/assets/writeups/DefconRTVCTF/6918cefa1bf44348b543537c2c2fc87c.png)

And then after messing around enough we realize there's no way to load actually it, so this was likely a troll...

So how about gdb?

![11432635340eb2a50137858ed0ff6499.png](/assets/writeups/DefconRTVCTF/8832d87e9e3c43679d780489690f3ae3.png)

![f27482c7910d7cf456b1bb6dc3045643.png](/assets/writeups/DefconRTVCTF/65e1651442194166be715525fbb61494.png)

Scrolling through, secret_function sure looks interesting.  So lets run the program, then ctrl-c to break during execution:

![dc88f513274d36c043f5d35907169f13.png](/assets/writeups/DefconRTVCTF/b232d90135e84e22944b2449ad5b5b19.png)

Now lets try to jump to the secret_function and see what happens:

![c4ff3d4b541361ff5fc8154d15c02a2d.png](/assets/writeups/DefconRTVCTF/e1f5db3716a441b2a017311fa0298d97.png)




