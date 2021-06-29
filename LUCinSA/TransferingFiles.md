(Transfering)=
# Transfering Files to/from cluster
================================================================================================================================

## via command line

To transfer small files via the command line, use <span style='color:Red'> rsync </span> . You can [learn more about rsync here](https://www.digitalocean.com/community/tutorials/how-to-use-rsync-to-sync-local-and-remote-directories)

#To Download a file from ERI to local:
```
rsync -raz --progress <username>@ssh.eri.ucsb.edu:<ERI path/fileName.ext> <local path/fileName.ext>
```
#To Upload a file from local to ERI
```
rsync <local path/fileName.ext> -raz --progress <username>@ssh.eri.ucsb.edu:<ERI path/fileName.ext>
```

## via FTP client
You can also transfer files using a File Transfer Protocol (FTP). If you are already using MobaXterm, you have an FTP included. Just double-click files from the file tree on the left to open them, or drag them onto your computer to copy them. You can also use a dedicated FTP such as WinSCP. ([Get WinSCP here](https://winscp.net/eng/index.php)).
As with SSH, you must be logged into a UCSB VPN client first.
| Step 1:                                | Step 2:                                |
| :------------------------------------: | :------------------------------------: |
|   ![](/Images/WinSCP2.png)             |  ![](/Images/WinSCP3.png)              |
| 1 - In "New Site", enter username and address for bellows. Then click "Advanced" | 2 - In "Tunnel", enter unsername and address for ERI. Click "OK" and save config. |
You will be prompted to enter your password twice.
Now you have a visual directory map from which you can transfer files between your PC and the Cluster

```{figure} /Images/WinSCP4.png
:scale: 20%
:name: WinSCP4.png
```