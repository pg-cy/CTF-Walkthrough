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

- So it looks like its using the OS ping command, which we can try to command stack using   `; && ||`
to get a reverse shell
 
