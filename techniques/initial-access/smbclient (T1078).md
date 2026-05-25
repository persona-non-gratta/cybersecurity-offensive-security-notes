Tool for comfortable interaction with the SMB protocol. 

``` shell
smbclient -NL <ip address>  # null-session (if allowed)
smbclient -U <username(%password)> <ip address>  # default session
```

``` shell
smbclient -NL <ip address> -c "one of the commands above" #one-line execution
```

```bash
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