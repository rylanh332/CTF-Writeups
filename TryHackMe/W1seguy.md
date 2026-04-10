# W1seguy

This challenge starts by giving us some source code to look at:

```
import random
import socketserver 
import socket, os
import string

flag = open('flag.txt','r').read().strip()

def send_message(server, message):
    enc = message.encode()
    server.send(enc)

def setup(server, key):
    flag = 'THM{thisisafakeflag}' 
    xored = ""

    for i in range(0,len(flag)):
        xored += chr(ord(flag[i]) ^ ord(key[i%len(key)]))

    hex_encoded = xored.encode().hex()
    return hex_encoded

def start(server):
    res = ''.join(random.choices(string.ascii_letters + string.digits, k=5))
    key = str(res)
    hex_encoded = setup(server, key)
    send_message(server, "This XOR encoded text has flag 1: " + hex_encoded + "\n")
    
    send_message(server,"What is the encryption key? ")
    key_answer = server.recv(4096).decode().strip()

    try:
        if key_answer == key:
            send_message(server, "Congrats! That is the correct key! Here is flag 2: " + flag + "\n")
            server.close()
        else:
            send_message(server, 'Close but no cigar' + "\n")
            server.close()
    except:
        send_message(server, "Something went wrong. Please try again. :)\n")
        server.close()

class RequestHandler(socketserver.BaseRequestHandler):
    def handle(self):
        start(self.request)

if __name__ == '__main__':
    socketserver.ThreadingTCPServer.allow_reuse_address = True
    server = socketserver.ThreadingTCPServer(('0.0.0.0', 1337), RequestHandler)
    server.serve_forever()
```

Looking through it we can see it sets up a flag and then xor's it against a provided key in the section here:

```
for i in range(0,len(flag)):
    xored += chr(ord(flag[i]) ^ ord(key[i%len(key)]))
```

When we nc into the IP using the port provided in the description we get this:

<img width="929" height="47" alt="image" src="https://github.com/user-attachments/assets/76b3240d-4c7f-4b75-96a1-f1d3a29aa88e" />

So we will be given a flag and then we will have to input the encryption key that created it. So it looks like we'll be doing a bit of reverse engineering

We know that the key is created using this line: `res = ''.join(random.choices(string.ascii_letters + string.digits, k=5))`

And we also know that the flag will start with THM{ which should cut out compute time for brute forcing it.

(NEEDS COMPLETEING)
