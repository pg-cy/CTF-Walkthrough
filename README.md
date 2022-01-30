# KIOPTRIX level1--Walkthrough


### Initial Port-Scan
- I usually like to scan the entire machine just incase they like to be sneaky and open a service on a random port number.
- port 139 was open so i tried to access the shares but couldnt, kept getting denied. Used smbmap,enum4linux and tried connecting directly to the port using smbclient which didnt provide much help.

![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/Images/nmapscan.png "network scan")
---------------
- From here i like to search for version exploits because some are easy to spot right away, especially if it looks really outdated.


