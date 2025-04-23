Transferring files
==================

To upload and download data you can use various methods, such as scp, sftp, and rsync. eEach method is described below. These instructions assume you are on VPN. If you are not, please see the orca user guide (LINK HERE) for instructions on how to transfer files through orca.

- [scp](#scp)
- [sftp](#sftp)
- [rsync](#rsync)
- [smb](#smb)

The examples below show how to use these tools from the command line, but you can also use graphical tools, such as:

- WinSCP
- FileZilla
- Cyberduck

## scp

To **upload** files or directories from your computer to the cluster:

```
scp /path/to/source <username>@hpc-head-002:/path/to/destination
```

For example:

```
scp -r myfiles/ robtest1234@hpc-head-002:~/mystuff
robtest1234@hpc-head-002's password:
file2                                                            100%    5     0.3KB/s   00:00
file3                                                            100%    5     0.3KB/s   00:00
file1                                                            100%    5     0.3KB/s   00:00
```


To **download** from the cluster to your computer:
```
scp -r <username>@hpc-head-002:/path/to/source /path/to/destination
```

For example:
```
scp -r robtest1234@hpc-jobs-001:~/mystuff .
robtest1234@hpc-jobs-001's password:
file3                                                            100%    5     0.1KB/s   00:00
file2                                                            100%    5     0.1KB/s   00:00
file1                                                            100%    5     0.1KB/s   00:00
```    

## sftp

To **upload** files or directories from your computer to the cluster, use the `put` option. For example:

```
sftp robtest1234@hpc-head-002
robtest1234@hpc-head-002's password:
Connected to hpc-head-002.
sftp> put -r myfiles/
Uploading myfiles/ to /home/robtest1234/myfiles
Entering myfiles/
file2                                                            100%    5     0.3KB/s   00:00
file3                                                            100%    5     0.3KB/s   00:00
file1                                                            100%    5     0.2KB/s   00:00
sftp> exit
 ```   

To **download** files or directories from the cluster to your computer, use the `get` option. For example:
```
sftp robtest1234@hpc-head-002
robtest1234@hpc-head-002's password:
Connected to hpc-head-002.
sftp> get -r mystuff
Fetching /home/robtest1234/mystuff/ to mystuff
Retrieving /home/robtest1234/mystuff
file3                                                            100%    5     0.1KB/s   00:00
file2                                                            100%    5     0.1KB/s   00:00
file1                                                            100%    5     0.1KB/s   00:00
sftp> exit
```

## rsync

To **upload** files or directories from your computer to the cluster:
```
rsync -avP /path/to/source <username>@hpc-head-002:/path/to/destination
```

For example:
```
rsync -avP mystuff robtest1234@hpc-head-002:~/somedir
-----------------------------------------------------------
---  This is the Natural History Museum in London, UK.  ---
---  If you are not an authorised user GO NO FURTHER !  ---
---  If you have problems connecting, contact :         ---
---          ts-servicedesk@nhm.ac.uk                   ---
-----------------------------------------------------------
robtest1234@hpc-head-002's password:
sending incremental file list
mystuff/
mystuff/file1
            5 100%    0.00kB/s    0:00:00 (xfr#1, to-chk=2/4)
mystuff/file2
            5 100%    4.88kB/s    0:00:00 (xfr#2, to-chk=1/4)
mystuff/file3
            5 100%    4.88kB/s    0:00:00 (xfr#3, to-chk=0/4)

sent 281 bytes  received 77 bytes  79.56 bytes/sec
total size is 15  speedup is 0.04
```

To **download** files or directories from the cluster to your computer:
```
rsync -avP  <username>@hpc-head-002:/path/to/source /path/to/destination
```

For example:
```
rsync -avP robtest1234@hpc-head-002:~/somedir/mystuff .
-----------------------------------------------------------
---  This is the Natural History Museum in London, UK.  ---
---  If you are not an authorised user GO NO FURTHER !  ---
---  If you have problems connecting, contact :         ---
---          ts-servicedesk@nhm.ac.uk                   ---
-----------------------------------------------------------
robtest1234@hpc-head-002's password:
receiving incremental file list
mystuff/
mystuff/file1
            5 100%    4.88kB/s    0:00:00 (xfr#1, to-chk=2/4)
mystuff/file2
            5 100%    4.88kB/s    0:00:00 (xfr#2, to-chk=1/4)
mystuff/file3
            5 100%    4.88kB/s    0:00:00 (xfr#3, to-chk=0/4)

sent 85 bytes  received 309 bytes  87.56 bytes/sec
total size is 15  speedup is 0.04
```
  
## smb

You can transfer files via `smb` (also known as `samba` or `cifs`) via **File Explorer** in Windows or **Finder** in macOS. 

To do this via File Explorer, browse to the location of the workspace, such as `\\valentine\mbl\share\workspaces\groups\<folder_name>`.

![file-explorer-smb](images/file-explorer-smb.png)

Or you can use the command line, as described below. You may need to install `smbclient` first. On Ubuntu Linux you can do this by running `sudo apt install smbclient`.

To **upload** a file, connect to Valentine:
```
smbclient -U <username>@nhm.ac.uk \\\\valentine\\<share_name>
```

Change to the directory you want to upload the file to:
```
cd <destination_folder>
```

Upload the file:
```
put <file_name>
```
Check that the file has been uploaded:
```
ls
```

Exit the prompt:
```
exit
```

For example:
```
[robef3@HYB-rF9MA5WUwGJ test]$ smbclient -U robef3@nhm.ac.uk \\\\valentine\\mbl
Password for [robef3@nhm.ac.uk]:
Try "help" to get a list of possible commands.
smb: \> cd share\workspaces\groups\rob-project-2\
smb: \share\workspaces\groups\rob-project-2\> put myfile.txt
putting file myfile.txt as \share\workspaces\groups\rob-project-2\myfile.txt (0.0 kb/s) (average 0.0 kb/s)
smb: \share\workspaces\groups\rob-project-2\> ls
  .                                   D        0  Tue Apr 22 16:48:44 2025
  ..                                  D        0  Tue Apr 22 13:31:23 2025
  myfile.txt                          A        0  Tue Apr 22 16:48:44 2025

                245111980032 blocks of size 1024. 66813775872 blocks available
smb: \share\workspaces\groups\rob-project-2\> exit
```

To **download** a file, connect to Valentine:
```
smbclient -U <username>@nhm.ac.uk \\\\valentine\\<share_name>
```

Change to the directory you want to download the file from:
```
cd <origin_folder>
```
Download the file:
```
get <file_name>
```
Exit the prompt:
```
exit
```

For example:
```
[robef3@HYB-rF9MA5WUwGJ test]$ smbclient -U robtest1234@nhm.ac.uk \\\\valentine\\mbl
Password for [robtest1234@nhm.ac.uk]:
Try "help" to get a list of possible commands.
smb: \> cd share\workspaces\groups\rob-project-2\
smb: \share\workspaces\groups\rob-project-2\> get myfile.txt
getting file \share\workspaces\groups\rob-project-2\myfile.txt of size 0 as myfile.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
smb: \share\workspaces\groups\rob-project-2\> exit
```
