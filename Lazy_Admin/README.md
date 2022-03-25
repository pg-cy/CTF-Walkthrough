# TryHackMe--Lazy_Admin


###  Initial Scan
- We can see from the port scan theres a webserver on port 80 and ssh on port 22. Ill start directory scanning while browsing the webserver
![alt text](https://github.com/pg-cy/lazy_admin/images/nmap.png)

### Directory scanning webserver
![alt text](https://github.com/pg-cy/lazy_admin/images/webdirectory.png)

- while going through some of the websites, i found 2 login forums. While my scans were going on still i decided to try manual SQL injection
- Login Site--	 http://10.10.104.171/content/as/
- 2nd login site--  http://10.10.104.171/content/as/index.php


- Tried basic sql bypass, didnt work. Couldnt get a conditional response or some sort of error so i tried time delays with examples below
- `'+||+(SELECT+sleep(10))--+`	 	--post greSQL
- `'+'(SELECT+pg_sleep(10))'--`        	--mysql
- `'+||+(SELECT+dbms_pipe.receive_message(('a'),10)+FROM+dual)--`		-oracle 	
- Tried a few other databases with some variations, nothing worked even tried using SQLmap and nothing...deadend.
-------------
- My scan finished, so im looking at each directory one by one for anything useful.

- looking at the content directory "http://10.10.26.143/content/inc/", your IP will be different. We found a backup directory. 
![alt text](https://github.com/pg-cy/lazy_admin/images/mysqlback.png)

- Backups are always interesting, lets download and have a peak at the file.
![alt text](https://github.com/pg-cy/lazy_admin/images/mysql_file.png)

- I tried cat'ing  the file to read in terminal but it looked really messy. I opened the file in visual studio code and searched for 'pass'
------------
- I found what looks like creds, not sure which is the correct one, so ill try both accounts on each login forum.
- Will need to crack the hash first `42f749ade7f9e195bf475f37a44cafcb`. You can use crackstation (google it), or john whatever you prefer. Cracksation is faster so ill just see what it gives me.


- Hash cracked  `Password123`
![alt text](https://github.com/pg-cy/lazy_admin/images/mysql_file.png)

- Now ill try both creds on each login forum.  `admin:Password123`   `manager:Password123`
- Login Site--	 http://10.10.104.171/content/as/
- 2nd login site--  http://10.10.104.171/content/as/index.php
![alt text](https://github.com/pg-cy/lazy_admin/images/sql-file.png)

- WOOOOOOO!. These creds workd on this login site-your IP will be different---- "http://10.10.191.158/content/as/"  `manager:42f749ade7f9e195bf475f37a44cafcb`


- I spent some time posting anything i could on those panels. I eventually got a directory and file to show up on the server, "http://10.10.132.32/content/inc/ads/"
![alt text](https://github.com/pg-cy/lazy_admin/images/ads.png)
![alt text](https://github.com/pg-cy/lazy_admin/images/woo.png)

- So the server is executing the file, so we can post our reverse shell code here, and get the backened server to execute it when we visit the page
- I used a php-reservse-shell from our kali machine. If your using something else you can look online php reverse shells `/usr/share/laudanum/wordpress/templates/php-reverse-shell.php`
- edit the file, change the IP to your VPN IP and Port address
![alt text](https://github.com/pg-cy/lazy_admin/images/edit.png)
--------

- I copied the contents of the file and posted it in the ads panel body section, and title you can make up anything
![alt text](https://github.com/pg-cy/lazy_admin/images/rev.png)
- After posting, your file should appear in this folder
- http://10.10.132.32/content/inc/ads/
---------
- Start a ncat listener-  `nc -lnvp 443`	--this case im using my port 443 i specified in the php-reverse file
- Click on your file that you just created. You should have a reverse shell now! wooo!
- I quickly stabalize the shell, you can google how to. Quick rundown, on victim shell type: python3 -c 'import pty; pty.spawn("/bin/bash")' press CTRL+Z, then type:  stty raw -echo, Then type: fg and hit ENTER.
------------
### Priv esc
- First thing i like to do when getting a shell, is check the users privileges `sudo -l`, then check binaries/files
- We can see from the screenshot the user can run a file with perl. Looks like we found our priv esc.
![alt text](https://github.com/pg-cy/lazy_admin/images/sudoL.png)


- Lets have a look at the file. Looks like the perl script we can run as sudo, is executing another script  "copy.sh"
- Looking at the copy.sh file, its executing a reverse shell, which would be one way to get root, if we changed the IP/port and just cathed the root shell.
- Or we can just get the perlscript to execute copy.sh script with a bash shell inside, which will be quicker. Since we only have write access on the copy.sh we need to use that as our way in.
![alt text](https://github.com/pg-cy/lazy_admin/images/file1.png)


- We can overwrite the file with echo.  `echo "/bin/bash" > /etc/copy.sh`
![alt text](https://github.com/pg-cy/lazy_admin/images/overwrite.png)
- Now run the sudo command `sudo /usr/bin/perl /home/itguy/backup.pl`
- WoooOOoOoo, we have root now!
![alt text](https://github.com/pg-cy/lazy_admin/images/root.png)

