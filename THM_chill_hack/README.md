# Try_Hack_Me Chill Hack room

### Initial port scan
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/Screenshot_2022-02-21_13-35-27.png)

### looking at FTP
- FTP is open, so looks quickly have a look before i begin diving into the webserver on port 80
- `ftP <ip>`   When proompted the credentials are anonymous:anonymous
- So theres one file there called "note.txt". To download locally we use `mget note.txt`
![alt text](https://github.com/pg-cy/CTF-Walkthrough/blob/main/THM_chill_hack/images/note.png)
- So not sure where this comes into play yet, but looks like were going to need to bypass some filtering

### Moving onto webserver on port 80
- Before/while i start bruteforcing directories, ill visit the webpage to see what im working with.
- While Bruteforcing directories i found /secret..so this looks interesting.
- On the 'secret' webpage had a user input field which looks like its processing commands.

- A lot of what i tried was filtered: python, nc, perl, 

- I used gawk to gain a reverse shell

- sudo -l

 
