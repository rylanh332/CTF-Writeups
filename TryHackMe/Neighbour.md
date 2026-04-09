# Neighbour

## User Flag: 

**nmap** to see what services are up

<img width="353" height="54" alt="image" src="https://github.com/user-attachments/assets/c56b0cd2-53ef-4cc2-8f65-9b40ebe4ff3a" />

As we can see, **http** is open (the desc of the challenge also revealed this)

This leads us to a default login page with a username and password, two things to note here. Firstly, we're in index.php, so there may be some directory fuzzing we can do here.

<img width="162" height="24" alt="image" src="https://github.com/user-attachments/assets/53f2add1-ff06-4d1c-b3c6-0475c284e6a0" />

Secondly, there is some clear information at our disposal in the source page, someone left this amazing comment for us to work off of (the page even hints to look there).

<img width="839" height="32" alt="image" src="https://github.com/user-attachments/assets/a21cb8ca-b4e7-4427-8b5f-01501931c835" />

So we'll sign in with guest : guest, and we can find this:

<img width="1264" height="157" alt="image" src="https://github.com/user-attachments/assets/fb827dcb-3ecc-4a19-a6a3-88d597ae2737" />

And if we navigate to the URL, we can see there is a **user variable** we can specify

<img width="359" height="37" alt="image" src="https://github.com/user-attachments/assets/b61a5600-6425-43a5-a020-3b78c285057d" />

Let's see if they sanitize it by trying out the admin account they told us not to look at

<img width="847" height="199" alt="image" src="https://github.com/user-attachments/assets/0d21f1aa-f396-4e98-891f-5dc6ad8439c2" />

And there it is, pretty easy one.
