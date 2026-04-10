# Lo-fi

As always, I **nmap** to start:

<img width="176" height="48" alt="image" src="https://github.com/user-attachments/assets/34c8fd79-16db-4ebe-84fc-da8ed1ee7982" />

So this IP has HTTP and ssh open. Let us take a look at the webpage:

<img width="1139" height="615" alt="image" src="https://github.com/user-attachments/assets/1a8d36a9-a3a4-4dee-80be-e0b5d32cc956" />

So we have a youtube link, discography section, and a search bar. Each of the discography sections lead to a different queryable webpage in the URL:

```
<li><a href="/?page=relax.php">Relax</a></li>
<li><a href="/?page=sleep.php">Sleep</a></li>
<li><a href="/?page=chill.php">Chill</a></li>    
<li><a href="/?page=coffee.php">Coffee</a></li>
<li><a href="/?page=vibe.php">Vibe</a></li>
<li><a href="/?page=game.php">Game</a></li>
```

If we input something in the search it will also change to `http://10.64.172.245/?search=test`

I'm going to try some directory brute forcing since it's responsive to php pages:

`gobuster dir -u http://10.64.172.245/ -w /home/kali/Desktop/SecLists/Discovery/Web-Content/common.txt -x php`

<img width="412" height="196" alt="image" src="https://github.com/user-attachments/assets/a369f4c6-3045-43c0-bc15-244a469cf82b" />

No new info from that, the only thing I managed to learn is that when you try to query for index.php inside the page index it loops it forever and crashes the page.

If we run something like `../` into the page query, the video no longer loads, which means that we are traversing outside the bounds of the page

`http://10.64.172.245/?page=../../../../../`

<img width="1167" height="498" alt="image" src="https://github.com/user-attachments/assets/94e913b7-426c-4419-8d41-e327e53bcf64" />

If we keep climbing down the paths using `http://10.64.172.245/?page=../../../etc/passwd` eventually we get it to display /etc/passwd`

<img width="761" height="377" alt="image" src="https://github.com/user-attachments/assets/95a6a831-df16-43cd-baa0-2efd441605a0" />

So once we try out the classic flag.txt, we manage to find what we're looking for:

<img width="761" height="160" alt="image" src="https://github.com/user-attachments/assets/30b20d82-8373-4d44-a038-11adda4746a6" />

