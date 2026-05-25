### SMB CLIENT:
The Server Message Blocks (SMB service) allows anonymous access via guest authentication (through a null session):
```bash 
445/tcp  open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
SNIP
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user  
```

[**Server Message Block** ](https://en.wikipedia.org/wiki/Server_Message_Block)(by default on port 445) - operates on Application Layer of TCP/IP stack. Protocol is used for share files, printers, serial ports and different communication
between nodes on a network on Windows-Based Machines.

**What means  "anonymous access via guest authentication"? Why is it vulnerable?**
Guest Access means that the server allows **connection without valid credential** (without any password or login), typically mapping  the session (named null session) to the **Guest account.** **Allowing this kind of sessions is the typical misconfiguration of SMB service.**

## SMB Client - Initial Access & Cheat Sheet
SMB allows connection within null-session (in guest/anonymous mode). Using  `smbclient` with flags `-N` (for skipping authentication and enter as a guest) and `-L` (for list all available shares), we can see next output: 

```shell
smbclient -NL <ip address>                        
	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	backups         Disk      
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
SMB1 disabled -- no workgroup available

## Some useful commands 
smb:\> ls                  # list files
smb:\> dir                 # same as ls
smb:\> cd folder           # change remote directory
smb:\> pwd                 # show current remote path
smb:\> lcd /dir            # change LOCAL directory

smb:\> get file.txt                # download one file
smb:\> mget *.txt                  # download multiple files
smb:\> put localfile.txt           # upload one file
smb:\> mput *.txt                  # upload multiple files
smb:\> reget file.txt              # resume download

smb:\> mkdir newdir                # create new directory
smb:\> rmdir olddir                # remove any directory
smb:\> rm file.txt                 # delete file
smb:\> rename old.txt new.txt      # rename file

smb:\> allinfo file.txt            # display all info
smb:\> stat file.txt               # persmission check

#downloads the whole directory
smb:\> recurse ON                  
smb:\> prompt OFF
smb:\> mget *

```

Useful commands - see in: [smbclient cheat sheet](../(smbclient))
