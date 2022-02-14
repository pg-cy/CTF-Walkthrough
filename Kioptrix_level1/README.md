# KIOPTRIX level1--Walkthrough


### Initial Port-Scan
----
- I usually like to scan the entire machine just incase they like to be sneaky and open a service on a random port number.
- port 139 was open so i quickly tried to access the shares but couldnt, kept getting denied. Used smbmap,enum4linux and tried connecting directly to the port using smbclient which didnt provide much help.

![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Kioptrix_level1/Images/nmapscan.png "network scan")

**********

- From here i like to search for version exploits because some are easy to spot right away, especially if it looks really outdated.
- Using the command line searchsploit to look up "apache httpd" hoping there be an exploit for apache v1.3.20 but nothing stood out to me.
- I then used searchsploit to look up "apache mod_ssl" to look for exploits against apache mod_ssl v2.8.4

![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Kioptrix_level1/Images/searchsploit.png "searching exploit")

- From the screenshot it looks like there is an exploit available apache mod_ssl v2.8.7 or a lower remote buffer overflow. I will give this exploit a try. I copied the exploit "cp /usr/share/exploitdb/exploits/unix/remote/764.c" to another directory.

### Fixing the exploit
- This exploit was quite difficult to run at the start, had to do a lot of googling because i kept running into errors when trying to compile it. 
- First you need to download the ptrace-kmod.c, you can use wget to download.  "wget  https://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c" 
- Reading the mod_ssl exploit File (764.c) it says to compile using syntax  "gcc -o OpenFuck OpenFuck.c -lcrypto". When trying to compile it will give you errors. Before compiling again we need to make a few changes. First you need to add these two lines of code at the top of "764.c".
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Kioptrix_level1/Images/fixing_exploit1.png)
- The exploit will also try to download the ptrace-kmod.c from the Internet. Since were running the Kioptrix machine on our home network we do not want it to have access to the internet. So instead the exploit will download the ptrace-kmod.c from our own attack machine (hence why we downloaded the ptrace-kmod.c at the start).
- Replace this HTTP site with our own webserver. (We will use python to spin up a quick webserver to host our ptrace-kmod.c) 
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Kioptrix_level1/Images/replace.png)
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Kioptrix_level1/Images/fixing_exploit2.png)

### Running the exploit
- Now we can compile our exploit. "gcc  -o  exploit  764.c  -lcrypto"
- Trying to execute our new exploit will show instructions on proper syntax. It needs to know the apache version & operating system to use, along with the IP address, port, and -c 40-50.
- Looking at the options our exploit syntax is  "./exploit 0x6b  10.0.2.9   443  -c 40"
_____
- Now we have full access to the machine, with root privileges   
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Kioptrix_level1/Images/root.png)


