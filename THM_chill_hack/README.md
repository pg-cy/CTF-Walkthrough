# Try_Hack_Me Chill Hack room

### Initial port scan
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/Screenshot_2022-02-21_13-35-27.png)

### looking at FTP
- FTP is open, so looks quickly have a look before i begin diving into the webserver


- Before i start bruteforcing directories, ill visit the webpage to see what im working with

-Found a 'secret' directory which had a user input that displayed commands.
- A lot of what i tried was filtered: python, nc, perl, 

- I used gawk to gain a reverse shell

- sudo -l

 
