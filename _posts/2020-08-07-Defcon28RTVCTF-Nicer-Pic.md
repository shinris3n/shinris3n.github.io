---
layout: post
title:  "An Even Nicer Picture"
date:   2020-08-07 22:21:15 -0400
category: [writeups]
challenge-source: [Defcon 28 - Red Team Village CTF]
challenge-category: ['forensics']
tags: [Defcon28-RTVCTF]
author: shinris3n
---

Forensics - An Even Nicer Picture

Trying the same method from the "A Nice Picture" challenge first:

![1e4483d70dd40bd252283f7d4444bce5.png](/assets/writeups/DefconRTVCTF/08b5bad066ee42c898fe71a40532993b.png)

There doesn't appear to be anything inside, but the file is clearly even larger than the first, identical picture, so let's see what else we can do.  Exiftool doesn't reveal anything in the metadata, so how about Steghide?

![252bbc0187d77aae3eda19d1f4c37fe8.png](/assets/writeups/DefconRTVCTF/271a2b683ea2445f8d06ffabc17812c1.png)

Nothing appears to show up, but maybe there's a passphrase.  Using a tool called Stegcracker (installed via pip3; defaults to using /usr/share/wordlists/rockyou.txt):

![52494346a7fe98388403a804931e74b0.png](/assets/writeups/DefconRTVCTF/a867e674f6c74744b34747584fb0c232.png)

Inspecting the output:

![8da2e8f1c47650b285da9b3f90eee1bd.png](/assets/writeups/DefconRTVCTF/c7c245f7832440d9b1d88763dad2ba1f.png)

Since the file starts with PK, this is a zip file.  So we rename it and try to open it up, finding that its password protected.

![694dbe167dc00d63991af8443b59eaa3.png](/assets/writeups/DefconRTVCTF/40cccb9ff22445cc9ea763b0bb1c3b3b.png)

This part is actually the exact same as the "A Nice Picture" problem (including the password), but running through the steps again, we can use fcrackzip using rockyou.txt as the wordlist here as well:

![51b9ac376c538407a14313cdc67d0f02.png](/assets/writeups/DefconRTVCTF/b7c07f8cf88d4c8c82fc2041203906b0.png)







