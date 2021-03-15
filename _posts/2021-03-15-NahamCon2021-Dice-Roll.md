---
layout: post
title:  "Dice Roll"
date:   2021-03-15 00:48:00 -0400
category: [writeups]
challenge-source: [NahamCon CTF 2021]
challenge-category: ['cryptography']
tags: [NahamConCTF2021]
author: shinris3n
---
<h1></h1>
<h3><font color=FF3030>Challenge Text</font></h3>

```
Author: @JohnHammond#6971

When you have just one source of randomness, it's "a die", but when you can have muliple -- it's 'dice!'

NOTE: You are welcome to "brute force" this challenge if you feel you need to. ;)

Download the file below and press the Start button on the top-right to begin this challenge.
``` 

<h1></h1>
<h3><font color=FF3030>Solution</font></h3>

- Downloading the code, we can see that the program allows us to randomly generate a 32 bit integer that represents a dice roll, reset the random seed ("shake the dice"), or try to guess the outcome of the next dice roll.  

{% highlight python %}

#!/usr/bin/env python3

import random
import os

banner = """
              _______
   ______    | .   . |\\
  /     /\\   |   .   |.\\
 /  '  /  \\  | .   . |.'|
/_____/. . \\ |_______|.'|
\\ . . \\    /  \\ ' .   \\'|
 \\ . . \\  /    \\____'__\\|
  \\_____\\/

      D I C E   R O L L
"""

menu = """
0. Info
1. Shake the dice
2. Roll the dice (practice)
3. Guess the dice (test)
"""

dice_bits = 32
flag = open('flag.txt').read()

print(banner)

while 1:
        print(menu)

        try:
                entered = int(input('> '))
        except ValueError:
                print("ERROR: Please select a menu option")
                continue

        if entered not in [0, 1, 2, 3]:
                print("ERROR: Please select a menu option")
                continue

        if entered == 0:
                print("Our dice are loaded with a whopping 32 bits of randomness!")
                continue

        if entered == 1:
                print("Shaking all the dice...")
                random.seed(os.urandom(dice_bits))
                continue

        if entered == 2:
                print("Rolling the dice... the sum was:")
                print(random.getrandbits(dice_bits))
                continue

        if entered == 3:
                print("Guess the dice roll to win a flag! What will the sum total be?")
                try:
                        guess = int(input('> '))
                except ValueError:
                        print("ERROR: Please enter a valid number!")
                        continue

                total = random.getrandbits(dice_bits)
                if guess == total:
                        print("HOLY COW! YOU GUESSED IT RIGHT! Congratulations! Here is your flag:")
                        print(flag)
                else:
                        print("No, sorry, that was not correct... the sum total was:")
                        print(total)

                continue

{% endhighlight %}

- Since seeding is optional and *getrandbits* (which is a deterministic, pseudo-random generator) is used, we should be able to predict values.  Luckily, there is a tool called [randcrack](https://pypi.org/project/randcrack/){:target="_blank"} available that can use 32-bit *getrandbits* integer inputs to predict newly generated numbers with high accuracy.  It takes 624 integers for randcrack to be ready to predict new numbers.

- To that end, we can build a script to interact with the challenge connection, roll the dice via option #2 and feed the results into randcrack.  After this we just use randcrack to guess the next roll with option #3 and get the flag.

{% highlight python %}
import os
import sys
import struct
import socket
from randcrack import RandCrack

rc = RandCrack()

address = 'challenge.nahamcon.com'
port = 31699
buffer = 4096
tcpsocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcpsocket.connect((address, port))

# Get the initial headers
received_data = tcpsocket.recv(buffer).decode('utf-8')
print ('Received:', received_data)

# Check information
tcpsocket.send(b'0\n')
received_data = tcpsocket.recv(buffer).decode('utf-8')
print ('Received:', received_data)

# Shake the dice
tcpsocket.send(b'1\n')
received_data = tcpsocket.recv(buffer).decode('utf-8')
print ('Received:', received_data)

# Roll the dice enough for RandCrack to be able to work its magic
for i in range(624):
        tcpsocket.send(b'2\n')
        received_data = tcpsocket.recv(buffer).decode('utf-8')
        received_data_lines = received_data.splitlines()
        received_roll = int(received_data_lines[1])
        rc.submit(received_roll)
        #print ('Received Roll Number',i,':', received_roll)

# Get a prediction of the next roll from RandCrack
try:
        roll_prediction = rc.predict_randrange(0, 4294967294)
        print ("Roll Prediction: ", roll_prediction)
except:
        print ('Not enough rolls to predict - change range to 625.')
        exit()

# Try to guess the next roll and output the response
tcpsocket.send(b'3\n')
received_data = tcpsocket.recv(buffer).decode('utf-8')
print ('Received:', received_data)
roll_prediction_encoded = (str(roll_prediction) + '\n').encode('utf-8')
tcpsocket.send(roll_prediction_encoded)
received_data = tcpsocket.recv(buffer).decode('utf-8')
print ('Received:', received_data)

tcpsocket.close()
{% endhighlight %}


{% highlight bash %}
─(shinris3n㉿kodachi)-[~/sec/CTFs/Nahamcon_2021/dice_roll]
└─$ python3 roll_a_lot.py                                                                 
Received: 
              _______
   ______    | .   . |\
  /     /\   |   .   |.\
 /  '  /  \  | .   . |.'|
/_____/. . \ |_______|.'|
\ . . \    /  \ ' .   \'|
 \ . . \  /    \____'__\|
  \_____\/

      D I C E   R O L L


0. Info
1. Shake the dice
2. Roll the dice (practice)
3. Guess the dice (test)

> 
Received: Our dice are loaded with a whopping 32 bits of randomness!

0. Info
1. Shake the dice
2. Roll the dice (practice)
3. Guess the dice (test)

> 
Received: Shaking all the dice...

0. Info
1. Shake the dice
2. Roll the dice (practice)
3. Guess the dice (test)

> 
Roll Prediction:  131726453
Received: Guess the dice roll to win a flag! What will the sum total be?
> 
Received: HOLY COW! YOU GUESSED IT RIGHT! Congratulations! Here is your flag:
flag{e915b62b2195d76bfddaac0160ed3194}

0. Info
1. Shake the dice
2. Roll the dice (practice)
3. Guess the dice (test)
{% endhighlight %}
