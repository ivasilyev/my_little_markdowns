# Share folder with a VMware VM

## Host-side

- Select the virtual machine and select VM > Settings.
- On the Options tab, select Shared Folders.
- Select a folder sharing option.
- (Optional) To map a drive to the Shared Folders directory, select Map as a network drive in Windows guests.
- This directory contains all of the shared folders that you enable. Workstation Pro selects the drive letter.
- Click Add to add a shared folder.
- On Windows hosts, the Add Shared Folder wizard starts. On Linux hosts, the Shared Folder Properties dialog box opens.
- Browse to, or type, the path on the host system to the directory to share.
- If you specify a directory on a network share, such as D:\share, Workstation Pro always attempts to use that path. 
If the directory is later connected to the host on a different drive letter, Workstation Pro cannot locate the shared 
folder.
- Specify the name of the shared folder as it should appear inside the virtual machine and click Next.
- Characters that the guest operating system considers illegal in a share name appear differently when viewed inside 
the guest. For example, if you use an asterisk in a share name, you see %002A instead of * in the share name on the 
guest. Illegal characters are converted to their ASCII hexadecimal value.
- Select shared folder attributes.
- Click Finish to add the shared folder.
- The shared folder appears in the Folders list. The check box next to folder name indicates that the folder is being 
shared. You can deselect this check box to turn off sharing for the folder.
- Click OK to save your changes.

To view a specific shared folder, go directly to the folder by using the UNC path 
`\\vmware-host\Shared Folders\shared_folder_name`.

## Client-side (Windows)

- Start Windows Explorer.
- Navigate to My Computer or Computer.
- Run the command to map a network drive. 
- Option. 
- Description. 
- Windows Vista, Windows 7, Windows 8, Windows 10. 
- Click Map Network Drive. ...
- Select a drive to map.
- In the Folder field, type `\\vmware-host\Shared Folders\` .
- Click Finish.

## Client-side (Linux)

```
sudo apt-get install -y open-vm-tools

sudo mkdir -p /vmhgfs
sudo chmod -R 777 /vmhgfs

sudo vmhgfs-fuse .host:/ /vmhgfs/ -o allow_other -o uid=1000 
ls /vmhgfs/

printf "\n.host:/ /vmhgfs/ fuse.vmhgfs-fuse defaults,allow_other,uid=1000 0 0\n" \
| sudo tee -a /etc/fstab
# sudo nano /etc/fstab

sudo shutdown -r  now
```
