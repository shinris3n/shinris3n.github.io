---
layout: post
title:  "Roll For Initiative 1"
date:   2020-08-07 22:23:15 -0400
category: [writeups]
challenge-source: [Defcon 28 - Red Team Village CTF]
challenge-category: ['programming']
tags: [Defcon28-RTVCTF]
author: shinris3n
---

Programming - Roll for Initiative 1

When connecting to the server, the "Backdoors and Breaches" game begins and we need to "Roll for Initiative":

![8fcfc4d3c0aec11816b20352a7954d6b.png](/assets/writeups/DefconRTVCTF/1b6fbdca30804930a87a5666ff2fbe1a.png)

Experimenting a bit we find:

- An integer between 1 and 20 needs to be provided as an input.
- The number needs to match an expected value.
- If it does we're asked to "Roll again" and provide another input. We're told that doing this 10 times will result in a "prize" which we can hope is the flag.
- If it does not, we're told it does not match, what the number should have been and the connection is broken.
- The expected numbers and their order do not change when we re-establish a connection to the server.

Our code therefore needs to open a connection to the server, submit some values, keep track of the correct values, restart the connect when an incorrect value is submitted, and find the flag, which we know will probably have '{' in it.

{% highlight python %}

import os
import sys
import struct
import socket

address = '164.90.147.2'
port = 1234
buffer = 4096
data_to_send = ['14'] * 10
flag = ''
received_data = ''
while flag == '': 
        tcpsocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        tcpsocket.connect((address, port))
        while 'What did you roll?' not in received_data:
                received_data = tcpsocket.recv(buffer).decode('utf-8')
        #print ('Received:', received_data)
        index = 0
        while True:
                #print ('Sending:', data_to_send[index], 'Array Index: ', index)
                formatted_data_to_send = (str(data_to_send[index]) + '\n').encode('utf-8')
                tcpsocket.send(formatted_data_to_send)
                received_data = (tcpsocket.recv(buffer)).decode('utf-8')
                #print ('Received:', received_data)
                if received_data == '':
                        break
                elif 'valid' in received_data:
                        print ('Error!')
                        sys.exit(0)
                elif '{' in received_data:
                        flag = received_data
                        print ('Flag =', flag)
                        break
                elif 'Correct' in received_data:
                        #print ('GOT IT!')
                        index += 1
                else:
                        new_array_entry = received_data.lstrip('Sorry, the roll I was looking for was ')        
                        #print ('Array Entry Added:', new_array_entry, 'Index:', index)
                        data_to_send[index] = new_array_entry

{% endhighlight %}

![0989ccb25f75075faa483cf4bde6b738.png](/assets/writeups/DefconRTVCTF/eea3a0b7795f46d2ad0c16dc3b5a464e.png)

