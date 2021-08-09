---
layout: post
title:  "ASCII Roulette"
date:   2021-08-07 19:20:12 -0400
category: [writeups]
challenge-source: [Defcon 29 - Red Team Village CTF]
challenge-category: ['programming', 'crypto']
tags: [Defcon29-RTVCTF]
author: shinris3n
---

![36b0b02e2b149d78d49234508c1f96a6.png](/assets/writeups/DefconRTVCTF2021/fc87d99fca484a23a74aef239560c99f.png)


- This was the original source code, which will take some effort to understand:

```cpp
#define lucky_num 24
#define angry_stache }
#define happy_stache {
#define extra_fate >>
#define lesser_fate <<
#define less_fate <
#define gained_fate +
#define lost_fate -
#define absolute_fate ==
#define given_fate =
#define start main()
#define lucky_number (rand()%7)+1;
#define cat for
#define give cout
#define take cin
#define seven flag
#define luck int
#define lucky string
#include <iostream>
#include <string>
#include <stdlib.h>
#include <time.h>
using namespace std;

luck start
happy_stache
        lucky seven;
        give lesser_fate "Enter Any String to Make it Lucky: ";
        take extra_fate seven;
        srand(time(NULL));
        luck seven_rand given_fate lucky_number
        give lesser_fate "Your Lucky Numbers are: ";
        for(luck i given_fate 0; i less_fate seven.length(); i++)
        happy_stache
                luck seven_num given_fate seven[i];
                seven_num given_fate seven_num ^ seven_rand;
                if(i % 2 absolute_fate 0)
                happy_stache
                        seven_num given_fate seven_num gained_fate lucky_num;
                angry_stache
                else
                happy_stache
                        seven_num given_fate seven_num lost_fate lucky_num;
                angry_stache
                give lesser_fate seven_num lesser_fate " ";
        angry_stache
angry_stache
```

- Substituting the defines throughout the code to make it readable:

```cpp
#define lucky_num 24
#define angry_stache }
#define happy_stache {
#define extra_fate >>
#define lesser_fate <<
#define less_fate <
#define gained_fate +
#define lost_fate -
#define absolute_fate ==
#define given_fate =
#define start main()
#define lucky_number (rand()%7)+1;
#define cat for
#define give cout
#define take cin
#define seven flag
#define luck int
#define lucky string
#include <iostream>
#include <string>
#include <stdlib.h>
#include <time.h>
using namespace std;

int main()
{
        string flag;
        cout << "Enter Any String to Make it Lucky: ";
        cin >> flag;
        srand(time(NULL));
        int flag_rand = (rand()%7)+1;
        cout << "Your Lucky Numbers are: ";
        for(int i = 0; i < flag.length(); i++)
        {
                int flag_num = flag[i];
                flag_num = flag_num ^ flag_rand;
                if(i % 2 == 0)
                {
                        flag_num = flag_num + 24;
                }
                else
                {
                        flag_num = flag_num - 24;
                }
                cout << flag_num << " ";
        }
}
```

- This shows us that the decimal value of each ASCII character in the string received from the commmand line is first bitwise XOR'd with a random number between 1 and 8.  This random number is chosen right at the beginning, so does not change.  

- 24 is then added to the value of every even indexed character in the string and 24 is subtracted from the value of every odd indexed character in the string.

- Now, we just have to write a script to reverse this entire process on the encoded string we are provided in the challenge, brute forcing the random offset value:


```python
roulette_string = "123 81 124 74 150 29 127 94 126 88 143 72 114 29 143 66 129 88 126 86 148 66 123 81 73 24 82 96"

roulette_array = list(map(int, roulette_string.split(" ")))

print ("ASCII Roulette Array: ", roulette_array)

flag = ""
index = 0
rand_num = 0
while rand_num <= 8:
	for value in roulette_array:
		if (index % 2 == 0):
			flag_num = (value - 24) ^ rand_num
		else:
			flag_num = (value + 24) ^ rand_num
		flag = flag + chr(flag_num)
		index = index + 1
	if flag [0:4] == "flag":
		print ("Flag Found for rand_num =", rand_num, ":", flag)
	rand_num = rand_num + 1
	flag = ""
```

![93b0c4bcaa0b44b849500aee79fbbb98.png](/assets/writeups/DefconRTVCTF2021/ba25ebd461c44f1aa35a98957494569b.png)