---
layout: post
title:  "layers"
date:   2021-10-24 23:06:42 -0400
category: [writeups]
challenge-source: [BuckeyeCTF2021]
challenge-category: ['misc']
tags: [BuckeyeCTF2021, docker]
author: shinris3n
---

<h1></h1>
<h3><font color=FF3030> Challenge Text: </font></h3>

```console
Author: qxxxb

Check out my brand new docker repo https://hub.docker.com/r/qxxxb/layers

``` 
<h3><font color=FF3030> Solution: </font></h3>

- Navigating to the site provided we're provided with a docker image to pull:

![50e68d39a21a5e9a267b120edf7c8a4b.png](/assets/writeups/BuckeyeCTF2021/50e68d39a21a5e9a267b120edf7c8a4b.png)

- Checking out the image we discover that it seems as if a file named flag.png once existed inside, but no longer does:

```console
┌──(shinris3n㉿kodachi)-[~/sec/CTFs/BuckeyeCTF2021/misc/layers]
└─$ docker pull qxxxb/layers
Using default tag: latest
latest: Pulling from qxxxb/layers
a0d0a0d46f8b: Pull complete 
9a653c77c575: Pull complete 
5b35be6cf17d: Pull complete 
5895c1ac3aab: Pull complete 
Digest: sha256:a89678536727abc0fbfe693b19ac0f8454502351dc792dabbee47bc9ab7420b2
Status: Downloaded newer image for qxxxb/layers:latest
docker.io/qxxxb/layers:latest

┌──(shinris3n㉿kodachi)-[~/sec/CTFs/BuckeyeCTF2021/misc/layers]
└─$ docker images
REPOSITORY      TAG       IMAGE ID       CREATED         SIZE
qxxxb/layers    latest    0c01a25ae5b7   5 days ago      6.19MB

┌──(shinris3n㉿kodachi)-[~/sec/CTFs/BuckeyeCTF2021/misc/layers]
└─$ docker run -it qxxxb/layers
/ # whoami
root
/ # ls -al
total 72
drwxr-xr-x    1 root     root          4096 Oct 25 00:51 .
drwxr-xr-x    1 root     root          4096 Oct 25 00:51 ..
-rwxr-xr-x    1 root     root             0 Oct 25 00:51 .dockerenv
-rw-rw-r--    1 root     root           124 Oct 19 15:26 Dockerfile
drwxr-xr-x    2 root     root          4096 Aug 27 11:05 bin
drwxr-xr-x    5 root     root           360 Oct 25 00:51 dev
drwxr-xr-x    1 root     root          4096 Oct 25 00:51 etc
drwxr-xr-x    2 root     root          4096 Aug 27 11:05 home
drwxr-xr-x    7 root     root          4096 Aug 27 11:05 lib
drwxr-xr-x    5 root     root          4096 Aug 27 11:05 media
-rw-r--r--    1 root     root            36 Oct 19 15:26 message.txt
drwxr-xr-x    2 root     root          4096 Aug 27 11:05 mnt
drwxr-xr-x    2 root     root          4096 Aug 27 11:05 opt
dr-xr-xr-x  306 root     root             0 Oct 25 00:51 proc
drwx------    1 root     root          4096 Oct 25 00:51 root
drwxr-xr-x    2 root     root          4096 Aug 27 11:05 run
drwxr-xr-x    2 root     root          4096 Aug 27 11:05 sbin
drwxr-xr-x    2 root     root          4096 Aug 27 11:05 srv
dr-xr-xr-x   13 root     root             0 Oct 25 00:51 sys
drwxrwxrwt    2 root     root          4096 Aug 27 11:05 tmp
drwxr-xr-x    7 root     root          4096 Aug 27 11:05 usr
drwxr-xr-x   12 root     root          4096 Aug 27 11:05 var
/ # cat message.txt 
Sorry, the flag has been deleted :(
/ # cat Dockerfile 
FROM alpine:latest
COPY flag.png Dockerfile /
RUN rm flag.png
RUN echo "Sorry, the flag has been deleted :(" > /message.txt
/ # ^C
/ # exit
```

- Checking the image history, it doesn't seem as if we can't easily just revert to previous layers because we didn't build this ourselves locally:

```console
┌──(shinris3n㉿kodachi)-[~/sec/CTFs/BuckeyeCTF2021/misc/layers]
└─$ docker history qxxxb/layers
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
0c01a25ae5b7   5 days ago    /bin/sh -c echo "Sorry, the flag has been de…   36B       
<missing>      5 days ago    /bin/sh -c rm flag.png                          0B        
<missing>      5 days ago    /bin/sh -c #(nop) COPY multi:6b3bd56201fda03…   599kB     
<missing>      8 weeks ago   /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B        
<missing>      8 weeks ago   /bin/sh -c #(nop) ADD file:aad4290d27580cc1a…   5.6MB  
```  

- However, digging a bit deeper we notice that the previous layers appear to have some artifacts left over that we can find on our local machine in /var/lib/docker/overlay2/:

```console
┌──(shinris3n㉿kodachi)-[~/sec/CTFs/BuckeyeCTF2021/misc/layers]
└─$ docker inspect qxxxb/layers                                                                                             
[
    {
        "Id": "sha256:0c01a25ae5b745b06c68c7b870b848f327227e06feca8f121c105d3cc423ebc9",
        "RepoTags": [
            "qxxxb/layers:latest"
        ],
        "RepoDigests": [
            "qxxxb/layers@sha256:a89678536727abc0fbfe693b19ac0f8454502351dc792dabbee47bc9ab7420b2"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2021-10-19T15:26:49.240573536Z",
        "Container": "f6fb5543fac6f92af65af594f2547a659761eecfa1cafd720d631e6ca2bb0e1e",
        "ContainerConfig": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "echo \"Sorry, the flag has been deleted :(\" > /message.txt"
            ],
            "Image": "sha256:d6aa6df23fb1b311908fad5a8eb92626650ece786144bc04e6ca02a3807fb02e",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": null
        },
        "DockerVersion": "20.10.6",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh"
            ],
            "Image": "sha256:d6aa6df23fb1b311908fad5a8eb92626650ece786144bc04e6ca02a3807fb02e",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": null
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 6194599,
        "VirtualSize": 6194599,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/a53ad803fcada3d5c4c3ad52b01d540c8d8a9358a46662dede804860ca9eba93/diff:/var/lib/docker/overlay2/aa2bcbd0951a0e898cb73e412ab47c09d77fc753c0947022e1fb0edfcc3cc08a/diff:/var/lib/docker/overlay2/6486d308eb4271abb881399b0f032c264a0296ad7612a1a82b41488a52eab8bb/diff",
                "MergedDir": "/var/lib/docker/overlay2/b93b559c421b5ec21015b2339e968457bcdbd7703f8c4181c6403db220ba7d5c/merged",
                "UpperDir": "/var/lib/docker/overlay2/b93b559c421b5ec21015b2339e968457bcdbd7703f8c4181c6403db220ba7d5c/diff",
                "WorkDir": "/var/lib/docker/overlay2/b93b559c421b5ec21015b2339e968457bcdbd7703f8c4181c6403db220ba7d5c/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:e2eb06d8af8218cfec8210147357a68b7e13f7c485b991c288c2d01dc228bb68",
                "sha256:a6951987fab1e53365680416db4c728a89783aa2b8c39bd2879aabfcffab95d9",
                "sha256:3d2c9f04c82dbb90411e102ed7e2e49b412b03e2f0021a40d3af818e02cdd0f7",
                "sha256:50d11bbfd2f65f57406491bdd23baad9a62672b8080d243a590c37e7dc7eab73"
            ]
        },
        "Metadata": {
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]
```

- We know we are searching for flag.png in one of these directories.  Navigating to /var/lib/docker/overlay2/ we hit a bit of a minor roadblock that won't be a problem on our own machine:

```console
┌──(shinris3n㉿kodachi)-[~/sec/CTFs/BuckeyeCTF2021/misc/layers]
└─$ cd /var/lib/docker/overlay2/

┌──(shinris3n㉿kodachi)-[/var/lib/docker/overlay2]
└─$ ls                                                                                                                             
ls: cannot open directory '.': Permission denied
```
- Trying this with elevated permissions, then searching via find for flag.png yields:

```console
┌──(shinris3n㉿kodachi)-[/var/lib/docker/overlay2]
└─$ sudo ls -al /var/lib/docker/overlay2/                                                                                          
total 76
drwx-----x 19 root root 4096 Oct 24 20:51 .
drwx--x--x 14 root root 4096 Sep 18 15:58 ..
drwx-----x  3 root root 4096 Jun  5 17:36 0c8211388805bd110c14a5156da4abefe9237cd11050d179f6870f092de974a7
drwx-----x  4 root root 4096 Jun  5 17:36 1d678d5825ea88e14b70dab71f24b46fbff22643294b4be6a0672632f188e68f
drwx-----x  3 root root 4096 Oct 24 20:49 6486d308eb4271abb881399b0f032c264a0296ad7612a1a82b41488a52eab8bb
drwx-----x  4 root root 4096 Oct 24 20:51 7b89f934aa1d9b1286d9ff09037b8f4cd25f3116e9ebdd30aa89b5be18fb01cd
drwx-----x  4 root root 4096 Oct 24 20:51 7b89f934aa1d9b1286d9ff09037b8f4cd25f3116e9ebdd30aa89b5be18fb01cd-init
drwx-----x  4 root root 4096 Oct 24 20:52 85850942fb425900d8354347b0f0233a63ddd8c226d88400138a740323d43569
drwx-----x  4 root root 4096 Oct 24 20:51 85850942fb425900d8354347b0f0233a63ddd8c226d88400138a740323d43569-init
drwx-----x  4 root root 4096 Jun  5 17:44 9a3d5ee146d1f52b551c49977384f451e1f1a9fb9f7a52ea055a6439f6c5dfe8
drwx-----x  4 root root 4096 Jun  5 17:36 9a3d5ee146d1f52b551c49977384f451e1f1a9fb9f7a52ea055a6439f6c5dfe8-init
drwx-----x  4 root root 4096 Oct 24 20:49 a53ad803fcada3d5c4c3ad52b01d540c8d8a9358a46662dede804860ca9eba93
drwx-----x  4 root root 4096 Jun  5 17:36 a76762e01180981bc7f9cf2ce77627c151d1155c2d7fa023a44c467d09493cd7
drwx-----x  4 root root 4096 Oct 24 20:49 aa2bcbd0951a0e898cb73e412ab47c09d77fc753c0947022e1fb0edfcc3cc08a
drwx-----x  4 root root 4096 Jun  5 17:36 ae4f50325c02c2387e3b39a797391617a633d3bc4144c5bb0b188b98b1ed2ba8
drwx-----x  4 root root 4096 Oct 24 20:51 b93b559c421b5ec21015b2339e968457bcdbd7703f8c4181c6403db220ba7d5c
drwx-----x  4 root root 4096 Jun  5 17:36 f9b87476e0cbfa8f0fab138781000c9f3c802e3350017e9facb9316feb4d44b1
drwx-----x  4 root root 4096 Jun  5 17:36 f9beca8f465f8ed70023d2297d975c6cd6b5a1490df9cf46c91627d0bd470538
drwx-----x  2 root root 4096 Oct 24 20:51 l

┌──(shinris3n㉿kodachi)-[/var/lib/docker/overlay2]
└─$ sudo find -name "flag.png"
./aa2bcbd0951a0e898cb73e412ab47c09d77fc753c0947022e1fb0edfcc3cc08a/diff/flag.png
./a53ad803fcada3d5c4c3ad52b01d540c8d8a9358a46662dede804860ca9eba93/diff/flag.png
```
- Trying to copy these files over to our home directory and opening them up is only successful for one of the two files.  (Inspecting the one that failed, the flag.png file appears to have the "c" (Character Device / Special File) permission set, which is not readable).

```console
┌──(shinris3n㉿kodachi)-[~/sec/CTFs/BuckeyeCTF2021/misc/layers]
└─$ sudo cp /var/lib/docker/overlay2/aa2bcbd0951a0e898cb73e412ab47c09d77fc753c0947022e1fb0edfcc3cc08a/diff/flag.png ./flag1.png    

┌──(shinris3n㉿kodachi)-[~/sec/CTFs/BuckeyeCTF2021/misc/layers]
└─$ sudo cp /var/lib/docker/overlay2/a53ad803fcada3d5c4c3ad52b01d540c8d8a9358a46662dede804860ca9eba93/diff/flag.png ./flag2.png
cp: cannot open '/var/lib/docker/overlay2/a53ad803fcada3d5c4c3ad52b01d540c8d8a9358a46662dede804860ca9eba93/diff/flag.png' for reading: No such device or address

┌──(shinris3n㉿kodachi)-[~/sec/CTFs/BuckeyeCTF2021/misc/layers]
└─$ sudo ls -al /var/lib/docker/overlay2/a53ad803fcada3d5c4c3ad52b01d540c8d8a9358a46662dede804860ca9eba93/diff/                    
total 8
drwxr-xr-x 2 root root 4096 Oct 24 20:49 .
drwx-----x 4 root root 4096 Oct 24 20:49 ..
c--------- 1 root root 0, 0 Oct 24 20:49 flag.png
```
- Opening the file that successfully copied over (flag1.png) yields the flag:

![373e23d292d39718b55e9934358780e0.png](/assets/writeups/BuckeyeCTF2021/373e23d292d39718b55e9934358780e0.png)

- We can now clean up any docker containers that are still running along with the image:

```console
┌──(shinris3n㉿kodachi)-[~/sec/CTFs/BuckeyeCTF2021/misc/layers]
└─$ docker ps -a                                                                                                                  
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS                        PORTS     NAMES
6e978af16952   qxxxb/layers           "/bin/sh"                57 minutes ago   Exited (130) 56 minutes ago             thirsty_rubin
d57ae7d74a36   qxxxb/layers           "/bin/sh"                57 minutes ago   Exited (0) 57 minutes ago               strange_bell

┌──(shinris3n㉿kodachi)-[~/sec/CTFs/BuckeyeCTF2021/misc/layers]
└─$ docker rm thirsty_rubin                                                                                                       
thirsty_rubin 

┌──(shinris3n㉿kodachi)-[~/sec/CTFs/BuckeyeCTF2021/misc/layers]
└─$ docker rm strange_bell                                                                                                         
strange_bell

┌──(shinris3n㉿kodachi)-[~/sec/CTFs/BuckeyeCTF2021/misc/layers]
└─$ docker rmi qxxxb/layers
Untagged: qxxxb/layers:latest
Untagged: qxxxb/layers@sha256:a89678536727abc0fbfe693b19ac0f8454502351dc792dabbee47bc9ab7420b2
Deleted: sha256:0c01a25ae5b745b06c68c7b870b848f327227e06feca8f121c105d3cc423ebc9
Deleted: sha256:34019abf87be9b68f1c90cc76b3d0c0b1c44aeb562f7745a54e1a47caf94f310
Deleted: sha256:fb972cfd15deb064bddfb9b7ed3718a78539b569f7cb604d7472ba64910cf9c8
Deleted: sha256:6d6c9aade1a3240edecab64e3d7ab6eeef677ef9570992191d55ccaab1039e4e
Deleted: sha256:e2eb06d8af8218cfec8210147357a68b7e13f7c485b991c288c2d01dc228bb68
```                       