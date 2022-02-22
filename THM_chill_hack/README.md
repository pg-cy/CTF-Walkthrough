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
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/ls.png)
- WOOOO SUCCESS!, was able to execute it.
- I tried the bypass with the "which" command to see what binaries i can use to get a reverse shell but wierdly it wouldnt work? not sure why, luckily i found a workaround using echo and command expansion.
- `echo $(ls /usr/bin | grep python)` I tried to see if python was available first, and theres python3 available
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/python.png)
--------
- Alright lets try a python reverse shell now. You can easily search python reverse shell online, ill be using one from GTFOBins website.

### ~~~~~~~~~~~~~~~~~~~~~~~~~~IN PROGRESS~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

