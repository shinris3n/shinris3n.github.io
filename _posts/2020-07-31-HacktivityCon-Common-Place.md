---
layout: post
title:  "Common Place"
date:   2020-07-31 17:38:23 -0400
category: [writeups]
challenge-source: [HacktivityCon2020]
challenge-category: ['warmups']
tags: [HacktivityCon2020]
author: shinris3n
---

I initially scrubbed through the html and js source code, looked for cookies and such but I couldn't figure this one out until checking what initially seemed like just flavor text:


&nbsp;&nbsp;&nbsp;&nbsp;![adbfd379f7e68fbc4a98fab24525a952.png](/assets/writeups/HacktivityCon2020/edc29275958045a89db03465980a3e49.png)

"rfc5785" found the flag...eh?

https://tools.ietf.org/html/rfc5785


&nbsp;&nbsp;&nbsp;&nbsp;![1e07aa6c1df1cf00e65310aeb3ad64f9.png](/assets/writeups/HacktivityCon2020/c704d87c2b1a4899ac021b4fd007d4b7.png)

And sure enough:

&nbsp;&nbsp;&nbsp;&nbsp;![3e7b8aee28d8aa0b69194cc4f97a2ba3.png](/assets/writeups/HacktivityCon2020/4f417fd06dae4c628afa19166ff2eb5b.png)

Dirbuster or Gobuster probably would have worked just fine also, and that was about to be my next step, but this was definitely more fun.


