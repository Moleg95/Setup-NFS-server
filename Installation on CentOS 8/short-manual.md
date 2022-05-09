# Installation packages on server

Run the command to install the package on the server with NFS:

***$ dnf install nfs-utils***

# Start service

Once the installation is complete, start the nfs-server service and enable it to automatically start at system boot:

 ***$ systemctl start nfs-server***

# Configuration NFS server
The configuration files for the NFS server are:

  */etc/nfs.conf* – main configuration file for the NFS daemons and tools.

  */etc/nfsmount.conf* – an NFS mount configuration file.

Next, create the file systems to export or share on the NFS server:

 ***$ mkdir  -p /mnt/commonfld***

Then export the above file systems in the NFS server */etc/exports* configuration file to determine local physical file systems 
that are accessible to NFS clients.

 ***/mnt/commonfld				192.168.30.6/24(rw,sync,no_all_squash,no_root_squash)***

   here are some of the exports options:
   
   192.168.30.6/24 - the address NFS client, if use "*" - all clients can use this folder
   
       - rw – allows both read and write access on the file system,
   
       - sync – tells the NFS server to write operations (writing information to the disk) when requested (applies by default),
   
       - all_squash – maps all UIDs and GIDs from client requests to the anonymous user,
   
       - no_all_squash – used to map all UIDs and GIDs from client requests to identical UIDs and GIDs on the NFS server,
   
       - root_squash – maps requests from root user or UID/GID 0 from the client to the anonymous UID/GID.

To export the above file system, run the command "exportfs":

 ***$ exportfs -arv***

options are:

   a - flag means export or unexport all directories, 
   
   r - means reexport all directories, synchronizing *var/lib/nfs/etab* with */etc/exports* and files under */etc/exports.d*,
   
   v - enables verbose output.

To display the current export list, run the following command:

 ***$ exportfs -s***

If the firewalld service running, need to allow traffic to the NFS services (mountd, nfs, rpc-bind) via the firewall, 
then reload the firewall rules to apply the change:

 ***$ firewall-cmd --permanent --add-service=nfs***
 
 ***$ firewall-cmd --permanent --add-service=rpc-bind***
 
 ***$ firewall-cmd --permanent --add-service=mountd***
 
 ***$ firewall-cmd --reload***

# Set up NFS client
On the client nodes, install the packages to access NFS shared folder. Run the command:
 
 ***$ dnf install nfs-utils nfs4-acl-tools***

Run the "showmount" command to show mount information for the NFS server. The command should output the exported file 
system on the client:

 ***$ showmount -e 192.168.30.231***
 
   192.168.30.231 - address of NFS server.
   
Create a local file system/directory on NFS client machine for mounting the remote NFS file system and mount it as nfs file system:

 ***$ mkdir -p /mnt/common***
 
 ***$ mount -t nfs  192.168.30.231:/mnt/commonfld /mnt/common***

To enable the mount to persistent after a client system reboot, run the command to write the entry in the /etc/fstab:

  ***$ echo "192.168.30.231:/mnt/commonfld  /mnt/common  nfs  defaults 0 0" >> /etc/fstab***
 
The last step - test, if NFS setup is working fine, create a file on the server and check if the file can be seen in the client 
and then do the reverse.

