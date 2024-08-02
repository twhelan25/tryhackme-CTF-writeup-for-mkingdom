![intro](https://github.com/user-attachments/assets/352398d3-7036-426f-8735-18b607e1f22e)

# tryhackme-CTF-writeup-for-mkingdom

This is a walkthrough for the tryhackme CTF mkingdom. I will not provide any flags or passwords as this is intended to be used as a guide.

## Scanning/Reconnaissance

First off, let's store the target IP as a variable for easy access.

Command: export ip=xx.xx.xx.xx

Next, let's run an nmap scan on the target IP:
```bash
nmap -A -v $ip -D RND:10 -oN nmap.txt
```

Command break down:

-A: This flag enables aggressive scanning. It combines various scan types (like OS detection, version detection, script scanning, and traceroute) into a single scan.

-v: increases verbosity, providing more detailed output during the scan.

—$ip: provides the target IP we stored as the variable $ip.

-D RND:10 Nmap can send additional packets to confuse network intrusion detection systems (IDS) or hide the true source of the scan by randomly selecting up to 10 decoy IP addresses.

-oN nmap.txt: This option specifies normal output that should be saved to a file named “nmap.txt.

This scan reveals a webserver on port 85:

![nmap](https://github.com/user-attachments/assets/6e9260bd-2c0f-4f72-ae61-0feece3a008a)

Visiting this webserver we see this image of Bowser:

![bowser](https://github.com/user-attachments/assets/5027b68b-1510-432f-8e82-b2b9824a84b5)

Using exiftool on the image reveals that the creater is b0ws53r. 

![exif](https://github.com/user-attachments/assets/6a372813-6216-499a-ad9f-8851eb1d1d98)



