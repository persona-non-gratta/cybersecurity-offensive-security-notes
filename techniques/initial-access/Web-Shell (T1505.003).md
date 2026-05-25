---
tags:
  - initial_access
  - web
---

#### What is the web-shell?
Web-shell is a file that will be parsed (executing via URL) and executed as code by webserver (user: `www-data` ). They are designed to give their users a means of executing arbitrary commands on the web-server. They are placed on the web-server without the permission of the owners of the site.

Typically, they work using `system()` or `exec()` functions. This is not by itself a vulnerability, but **if an attacker have access to them, or can control passed parameters** ( like with `templateselection`) it becomes a severe-vulnerability, which can lead to: Remote Code Execution (**RCE); File Upload; File Download; Database compromising, etc..**

[CVE-2025-64714 (GHSA-g2j9-g8r5-rg82)](https://nvd.nist.gov/vuln/detail/CVE-2025-64714) tells us about vulnerable cookie parameter - `template`, which we can exploit (since `templateselection` is enabled ) using curl, triggering our payload.
``` bash
~ curl -k https://bin.kobold.htb/ \
  -b "template=../data/pwn" \ 
  -G --data-urlencode "cmd=whoami"
nobody
```
In our case, PrivateBin with enabled `templateselection` trust the values `tempalte` and directly inserts it in the `include()` function - without any validation - it allows us to transmit to the server path to our new-created PHP payload, instead of legitimate template.

``` bash
-b #specifies a cookie parameter
-G --data-urlencode #get method + adding data with url encoding
```
We got response - `nobody`! It's our session's id, which means we are in the system.