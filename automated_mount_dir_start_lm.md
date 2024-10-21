## Automated mounting of NAS-directories during the start of Linux Mint

**Content**  
How to mount NAS-directories (NAS: network attached storage) during the start of your operating system like Linux Mint (LM) in a local area network (LAN) to a specified subdirectory "/mnt/NAS/NAS_directory" of LM.

**Acknowledgement**  
Most parts of the sources for this tutorial were generated in multiple sessions with the assistance of ChatGPT, an AI language model developed by OpenAI. Contributions of ChatGPT were modified due to didactic reasons.

**Disclaimer**  
The author is responsible for the contents of this tutorial but not for data losses or corruption of your LM due to the application of this tutorial. Assistance for the usage of this tutorial can't be provided due to missing capacities.

**Preliminary remarks**  
The described process below works only when the NAS is in operation and accessible at the start of LM.
The commands below are performed via the terminal of LM, exceptions are explicitly mentioned.
The mounting of a NAS directory in a protected private network is chosen to avoid security issues and recommended for users without experience in this area - "from simple to difficult".
In order to restrict the length of this tutorial, the explanations have been limited to the necessary steps.

**Steps - Overview**

1. Prepare the NAS and LM
2. Create a protected file with user login data
3. Backup and enhance "fstab"
4. Create a mount directory in LM
5. Test the enhanced "fstab"
6. Troubleshooting - possible issues
7. Post processing
8. Usage of alternative file services

**Steps in detail**

1. Prepare the NAS and LM  
   For mounting NAS-directories the file sharing protocol Server Message Block (SMB) is recommended because it is widely supported across Windows, Linux, and macOS, making it versatile for mixed environments. Others advantages are better security features, including encryption and authentication methods and a rich feature set. Disadvantages of this service are the bigger overhead than other file sharing protocols, which may impact performance in certain scenarios, a more complex configuration, especially when integrating with non-Windows systems, and firewall issues due to multiple ports being used.

   In LM the usage of SMB requires installed packages samba, cifs-utils and smbclient. To verify that these packages are installed type in the terminal: dpkg -l | grep samba, dpkg -l | grep cifs-utils, dpkg -l | grep smbclient
   These commands will show the versions and installation status of the respective packages. If nothing is returned, the respective package is not installed.

   You should use the latest versions of SMB that are supported by LM and your NAS because of improved functionality and security mechanisms.
2. Create a protected file with user login data  
   Create the file: /etc/samba/credentials: sudo xed /etc/samba/credentials
   Add login data:
   username=your_username
   password=your_password

   Save the file and protect it: sudo chmod 600 /etc/samba/credentials
   "chmod 600" sets file permissions so that only the owner can read and write the file. This will be sufficient for a protected LAN.
3. Backup and enhance "fstab"  
   In the terminal: sudo cp /etc/fstab /etc/fstab.bak
   The file fstab (located at /etc) is a configuration file that defines how disk partitions, devices, and other file systems are mounted into the filesystem structure. It specifies the device name, mount point, file system type, mount options, dump options, and pass number for file system checks.

   Open the existing fstab with privileged rights in xed: sudo xed /etc/fstab
   Add a comment at the end of fstab like: # user defined NAS mappings
   Mount a NAS-directory: //192.168.XXX.YYY/NAS_directory /mnt/NAS/NAS_directory cifs credentials=/etc/samba/credentials,iocharset=utf8,vers=3.0,uid=your_username,gid=your_username, file_mode=0777,dir_mode=0777,nounix 0 0
   Save fstab in xed.

   Start with one directory to verify the new mounting process and firstly set all user privileges.
   The mount options file_mode / dir_mode=0777 in fstab set the file and directory permissions when mounting a SMB share. It means full read, write, and execute permissions for the owner, group, and others.
4. Create a mount directory in LM  
   Create the needed directory on LM: sudo mkdir -p /mnt/LM-directory
   "/mnt/..." is chosen for the mapping because it is traditionally used for temporarily mounting filesystems by the system administrator. Opposed to that the directory "/media/..." is used for automatically mounting removable media (like USB drives, CDs, etc.) by desktop environments and utilities. Mounted directories on "/media/..." are displayed on the desktop of LM.
5. Test the enhanced "/etc/fstab"  
   Check the fstab on syntax errors: sudo mount -aAfter activating the command no error message should appear and the respective NAS directory should be mounted.
   Verify the correct mounting: df -h
   The command before lists all mounted file systems - inclusive the mounted NAS-directory.
   Check the user rights of the mounted directory: ls -l /mnt/NAS_directory
   Open some documents of the mounted directory / directories to check the file permissions: creating / deleating / reading / renaming / writing.
   User rights are mainly defined with the help of the NAS operating system. Some NAS operating systems offer the option to use the standard UNIX user rights which isn't useful because these rights represent only simple alternatives.
6. Possible issues - Troubleshooting  
   Linux Mint, NAS and Router: Firewall(s) block(s) connections between the NAS and LM
   Firstly disable the firewalls of LM and the NAS. Continue the tests, in case of success enable them to find out where the necessary modifications have to be done.
7. Post processing  
   Observe for a few days the established process, edit files of the first mounted directory for all of your typical use cases and adapt the necessary user privileges by adapting the the command in fstab if necessary.
   If the access rights cover all of your use cases mount additional directories by appending similar commands in fstab.
8. Usage of alternative file services    

**NFS (Network File System)**
NFS is a distributed file system protocol originally developed for Unix/Linux systems, allowing remote access to files over a network as if they were local. A lot of Internet sources mention that NFS seems to be faster depending on the sizes and number of files to be shared. The main disadvantages are that it is less secure by default compared to SMB, it requires a careful configuration to ensure security, it possesses a limited native support on non-Unix systems, although clients exist, and it can be complex to set up properly in heterogeneous environments.
Although the NFS service is masked in LM nevertheless a mounting of NAS directories on the basis of NFS works.

In LM open a terminal and check whether NFS is installed: dpkg -l | grep nfs  
Mount a NAS-directory in fstab: //NAS-IP/NAS_directory /mnt/NAS_directory nfs rw,noatime,nfsvers=4.1 0 0

**WebDAV (Web Distributed Authoring and Versioning)** - for the sake of completeness, not tested
It is an extension of the HTTP protocol that allows users to collaboratively edit and manage files on remote Web-servers. It provides a way to access files over the Web in a manner similar to local file systems. You should use it when your data are located on a Web-server. One advantage is that it works with Web-servers and can be accessed from various operating systems and devices. Disadvantages are that it can be slower than NFS or SMB, and it requires a proper configuration for secure access (e.g. using HTTPS) to avoid vulnerabilities.

For mounting NAS directories in LM you typically need a WebDAV-client provided by the respective package "davfs2" or using the built-in file manager with WebDAV support. Similar to SMB there is a need for access credentials like a username and password if the WebDAV-server requires authentication. The mount point in LM may be a local directory where the WebDAV share will be mounted (e.g. "/mnt/webdav").
