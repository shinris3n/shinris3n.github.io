---
layout: post
title:  "N1TP"
date:   2021-09-18 18:25:21 -0400
category: [writeups]
challenge-source: [HacktivityCon2021]
challenge-category: ['cryptography']
tags: [HacktivityCon2021]
author: shinris3n
---

<h1></h1>
<h3><font color=FF3030> Challenge Text: </font></h3>

```console
Author: @JohnHammond#6971

Nina found some new encryption scheme that she apparently thinks is really cool. She's annoying but she found a flag or something, can you deal with her?

Connect with:

nc challenge.ctf.games 31921
``` 
<h3><font color=FF3030> Solution: </font></h3>

- Connecting to the service, we are provided with the encrypted flag and allowed to encrypt any string we like.  For this competition, we know that the flag is 32 characters in the format "flag{0-9a-f}", so we can check out what "flag{" maps to:

```console
┌──(shinris3n㉿kodachi)-[~]
└─$ nc challenge.ctf.games 31921
NINA: Hello! I found a flag, look!
      2b372f6747ea6adc33cd0f71ced9f039c3d803dd36c610eddc16007d63b73d1cd2c5876a01d8
NINA: But I encrypted it with a very special nonce, the same length as 
      the flag! I heard people say this encryption method is unbreakable!
      I'll even let you encrypt something to prove it!! What should we encrypt?
> test
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?
      393e3d7448b62b9f71cb18678d8ae27ed3d946cb73c556ab9b41433074b6770b96c1c52f17c0
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> flag{
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?
      2b372f6747b5348a62d50d7f9888ea6ccbdd52c461cc44b894425c2567a8621383c3cd3d0fc4
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
```
- The output does indeed seem to indicate that a one time pad is being used... except it is the same pad being used for every string we enter.  It also appears that each character we enter is mapped to a hexadecimal value.  We can assume these things because we input a five character string and saw that the first 10 characters of the encrypted string we received back were the same as the first 10 characters of the flag we were provided when we connected to the service.  

- If we disconnect from the service and re-connect, however, the pad appears to change: 

```console
┌──(shinris3n㉿kodachi)-[~]
└─$ nc challenge.ctf.games 31921
NINA: Hello! I found a flag, look!
      11fdfea632c2fe3807fcbdc68360e71166db3e57509c9880fba21cbb35f24fe5e68728d5cf00
NINA: But I encrypted it with a very special nonce, the same length as 
      the flag! I heard people say this encryption method is unbreakable!
      I'll even let you encrypt something to prove it!! What should we encrypt?
> test
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?
      03f4ecb53d9ebf7b45faaad0c033f55676da7b41159fdec6bcf55ff622f305f2a2836a90d918
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> flag{
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?
      11fdfea6329da06e56e4bfc8d531fd446ede6f4e0796ccd5b3f640e331ed10eab7816282c11c
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
```

Therefore, we'll have to stay connected and interact with the service until we determine what the flag is.  The Python code below accomplishes this.

```python
# H@cktivityCon 2021 - N1TP Solution
# shinris3n

import os
import sys
import struct
import socket
import time

address = 'challenge.ctf.games'
port = 31921
buffer = 4096

# The encrypted flag received from "NINA" after connecting to the service.
encrypted_flag = ''

# The rules state: Flags for this competition will follow the format: flag\{[0-9a-f]{32}\}. That means a `flag{}` wrapper with what looks like an MD5 hash inside the curly braces.
input_to_encrypt = 'flag{' + '0'*32 + '}'

received_data = ''

possible_flag_characters = '0123456789abcdef'

# Open up connection.
tcpsocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcpsocket.connect((address, port))

# Get greeting.
received_data = tcpsocket.recv(buffer).decode('utf-8')
print (received_data)

# Retrieve and store the encrypted flag. 
received_data = tcpsocket.recv(buffer).decode('utf-8')
encrypted_flag = received_data[6:82] # The flag appears to be hex formatted, 64 characters plus 12 characters for the flag{} portion, and 6 spaces of padding.
print ('Encrypted Flag:', encrypted_flag)

# Send input to encrypt in flag format.
formatted_data_to_send = (input_to_encrypt + '\n').encode('utf-8')
tcpsocket.send(formatted_data_to_send)
print ('Data sent:', input_to_encrypt)

# Get and store encrypted response.
received_data = tcpsocket.recv(buffer).decode('utf-8') # Get flavor text
print (received_data)
received_data = tcpsocket.recv(buffer).decode('utf-8') # Get encrypted response
encrypted_response = received_data[6:82] # Same format as the original flag.
print (received_data)
print ('Encrypted Flag:', encrypted_flag)
print ('Encrypted Response:', encrypted_response)

# Now cycle through the possible flag characters, checking values until everything matches.
# Starting with character 6 (array index 5), cycle through possible flag characters and move on only after finding a match.
flag_match_index = 5

while True:
        
        for possible_flag_character in possible_flag_characters:
                input_to_encrypt = input_to_encrypt[:flag_match_index] +possible_flag_character + input_to_encrypt[flag_match_index + 1:]
                #time.sleep(2) # Two second delay to avoid overloading service.
                # Send input to encrypt in flag format.
                formatted_data_to_send = (input_to_encrypt + '\n').encode('utf-8')
                tcpsocket.send(formatted_data_to_send)
                print ('Data sent:', input_to_encrypt)
                received_data = tcpsocket.recv(buffer).decode('utf-8') # Get flavor text
                print (received_data)
                received_data = tcpsocket.recv(buffer).decode('utf-8') # Get encrypted response
                encrypted_response = received_data[6:82] # Same format as the original flag.
                print (received_data)
                print ('Encrypted Flag:', encrypted_flag)
                print ('Encrypted Response:', encrypted_response)        
                
                if encrypted_flag[flag_match_index*2] == encrypted_response[flag_match_index*2]:
                        if encrypted_flag[flag_match_index*2+1] == encrypted_response[flag_match_index*2+1]:
                                flag_match_index = flag_match_index + 1
                                print ('Matched Character:', input_to_encrypt[flag_match_index-1])
                                print ('Next Character to Match:', input_to_encrypt[flag_match_index])
                                if input_to_encrypt[flag_match_index] == '}':
                                        # Close out connection.
                                        tcpsocket.shutdown(1)
                                        tcpsocket.close()
                                        print ('Flag Found:' ,input_to_encrypt)
                                        exit(0)
```

- Below is a snippet of the corresponding output, along with the flag:

```console
┌──(shinris3n㉿kodachi)-[~/sec/CTFs/HacktivityCon2021/N1TP]
└─$ python3 ./n1tp.py 
NINA: Hello! I found a flag, look!

Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Data sent: flag{00000000000000000000000000000000}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4edadd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4edadd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
Data sent: flag{00000000000000000000000000000000}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4edadd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4edadd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
Data sent: flag{10000000000000000000000000000000}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4edbdd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4edbdd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
Data sent: flag{20000000000000000000000000000000}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4ed8dd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ed8dd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
Data sent: flag{30000000000000000000000000000000}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4ed9dd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ed9dd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
Data sent: flag{40000000000000000000000000000000}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4ededd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ededd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
Data sent: flag{50000000000000000000000000000000}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4edfdd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4edfdd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
Data sent: flag{60000000000000000000000000000000}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4edcdd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4edcdd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
Data sent: flag{70000000000000000000000000000000}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4edddd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4edddd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
Data sent: flag{80000000000000000000000000000000}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4ed2dd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ed2dd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
Data sent: flag{90000000000000000000000000000000}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4ed3dd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ed3dd9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
Matched Character: 9
Next Character to Match: 0
Data sent: flag{9a000000000000000000000000000000}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4ed38c9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ed38c9400de7e8f1196220a31c43460b13d7b95450b99513f8b44bc6d526aa8b0ea
Data sent: flag{9b000000000000000000000000000000}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

...
...
...

NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036bfeb0ea
Data sent: flag{9276cdb76a3dd6b1f523209cd9c0a100}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba8b0ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba8b0ea
Data sent: flag{9276cdb76a3dd6b1f523209cd9c0a110}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9b0ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9b0ea
Matched Character: 1
Next Character to Match: 0
Data sent: flag{9276cdb76a3dd6b1f523209cd9c0a112}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9b2ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9b2ea
Data sent: flag{9276cdb76a3dd6b1f523209cd9c0a113}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9b3ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9b3ea
Data sent: flag{9276cdb76a3dd6b1f523209cd9c0a114}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9b4ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9b4ea
Data sent: flag{9276cdb76a3dd6b1f523209cd9c0a115}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9b5ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9b5ea
Data sent: flag{9276cdb76a3dd6b1f523209cd9c0a116}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9b6ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9b6ea
Data sent: flag{9276cdb76a3dd6b1f523209cd9c0a117}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9b7ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9b7ea
Data sent: flag{9276cdb76a3dd6b1f523209cd9c0a118}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9b8ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9b8ea
Data sent: flag{9276cdb76a3dd6b1f523209cd9c0a119}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9b9ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9b9ea
Data sent: flag{9276cdb76a3dd6b1f523209cd9c0a11a}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e1ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e1ea
Data sent: flag{9276cdb76a3dd6b1f523209cd9c0a11b}
NINA: Ta-daaa!! I think this is called a 'one' 'time' 'pad' or something?

      735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
NINA: Isn't that cool!?! Want to see it again? 
      Sorry, I forget already -- what was it you wanted to see again?
> 
Encrypted Flag: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Encrypted Response: 735097cf4ed3df93068d2add1690730965903232b06b7e97460999586cdf4def6d036ba9e2ea
Matched Character: b
Next Character to Match: }
Flag Found: flag{9276cdb76a3dd6b1f523209cd9c0a11b}
```
