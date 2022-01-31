# KIOPTRIX level1--Walkthrough


### Initial Port-Scan
----
- I usually like to scan the entire machine just incase they like to be sneaky and open a service on a random port number.
- port 139 was open so i quickly tried to access the shares but couldnt, kept getting denied. Used smbmap,enum4linux and tried connecting directly to the port using smbclient which didnt provide much help.

![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Images/nmapscan.png "network scan")

**********

- From here i like to search for version exploits because some are easy to spot right away, especially if it looks really outdated.
- Using the command line searchsploit to look up "apache httpd" hoping there be an exploit for apache v1.3.20 but nothing stood out to me.
- I then used searchsploit to look up "apache mod_ssl" to look for exploits against apache mod_ssl v2.8.4

![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Images/searchsploit.png "searching exploit")

- From the screenshot it looks like there is an exploit available apache mod_ssl v2.8.7 or a lower remote buffer overflow. I will give this exploit a try. I copied the exploit "cp /usr/share/exploitdb/exploits/unix/remote/764.c" to another directory.

- This exploit was quite difficult to run at the start, had to do a lot of googling because i kept running into errors when trying to compile it. 
- First you need to download the ptrace-kmod.c, you can use wget to download.  "wget  https://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c" 
- Reading the mod_ssl exploit File (764.c) it says to compile using syntax  "gcc -o OpenFuck OpenFuck.c -lcrypto". When trying to compile it will give you errors. You need to add these two lines of code at the top of "764.c" file.
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Images/fixing_exploit1.png)

