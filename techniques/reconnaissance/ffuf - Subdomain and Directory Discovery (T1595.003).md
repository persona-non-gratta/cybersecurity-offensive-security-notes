---
tags:
  - enumeration
---

# Filtering (Before Scanning)
``` bash
curl -k https://kobold.htb -H "Host: subdomaincheck.kobold.htb" | wc -c
~ 154
```
#### Directory Enumeration
``` bash
 ffuf -w ~/Documents/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -u https://10.129.2.17/FUZZ -k -fs 154 -ac                                                             
```
#### Subdomain Enumeration
``` bash
ffuf -w ~/Documents/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt -u https://10.129.2.17 -H "Host: FUZZ.kobold.htb" -k -fs 154 -ac
```

``` bash
-w #specifying of the wordlist 
-u #our target url
-H + "Host: FUZZ.kobold.htb" #specifyng the header and the place we want fo FUZZ inside it 
-k #skipping certification check
-fs #reponse size filter
-ac #automatically callibrates filtering options (additional filtering)
```
```