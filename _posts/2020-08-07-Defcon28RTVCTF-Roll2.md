---
layout: post
title:  "Roll For Initiative 2"
date:   2020-08-07 22:24:15 -0400
category: [writeups]
challenge-source: [Defcon 28 - Red Team Village CTF]
challenge-category: ['programming']
tags: [Defcon28-RTVCTF]
author: shinris3n
---

Programming - Roll for Initiative 2

Just like in Roll for Initiative 1: when connecting to the server, the "Backdoors and Breaches" game begins and we need to "Roll for Initiative".  

![0bdce67876048f6436d744266c6158a1.png](/assets/writeups/DefconRTVCTF/3c488cb73c024d5cb17bb6d3b3de9ff7.png)

The functionality is exactly the same as well:

- An integer between 1 and 20 needs to be provided as an input.
- The number needs to match an expected value.
- If it does we're asked to "Roll again" and provide another input. We're told that doing this 10 times will result in a "prize" which we can hope is the flag.
- If it does not, we're told it does not match, what the number should have been and the connection is broken.
- The expected numbers and their order do not change when we re-establish a connection to the server.

However, now 100 "wins" (value matches) in a row is the goal.  This makes it a lot more tedious to manually keep track of the correct values and pretty much forces code to be written to get the flag.

The code written for the first problem is easily modified to work for this one also by changing the port number and array initialization size:

{% highlight python %}

import os
import sys
import struct
import socket

address = '164.90.147.2'
port = 1235
buffer = 4096
data_to_send = ['15'] * 100
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

![97cf50532cacd59f9000fbc66f663bb1.png](/assets/writeups/DefconRTVCTF/c9b2166cbc7b4c20aca78c8269129419.png)

