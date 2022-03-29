# Archangel Room


### Initial Port-scan
![alt-text]()

- There is a webserver on port 80, so ill directory bruteforce to see if i can find anything interesting.

![alt-text]()

- Looking at the Directory scan, theres a /flags folder. I knew this was too easy, when i clicked on the link inside the file, it links to a youtube video "rick-rolled". I searched through a bunch of other directories but nothing stood out.
--------------
- At the top of the page of the main site, we we can see another hostname, which is one of the questions THM was asking for. 
![alt-text]()

- Now to be able to reach that site, we need to specify the address in our /etc/hosts file. Youll need sudo to edit it.
![alt-text]()

- After we can just put in the host-name in the url and it will take us to that site.
- Ok since this is a completely new site lets start directory bruteforcing again with fereoxbuster.



