## NFS Server setup on "Ubuntu 24.04.3 LTS"

- install **NFS** on your designated **NFS Server**:  ````apt install nfs-kernel-server -y````

- Verify status of the **NFS service**: ```systemctl status nfs-server```

- once satus is running, you'll notice the following error: 

```
Dec 16 17:12:33 storageNode systemd[1]: Starting nfs-server.service - NFS server and services...
Dec 16 17:12:33 storageNode exportfs[1796]: exportfs: can't open /etc/exports for reading
Dec 16 17:12:33 storageNode systemd[1]: Finished nfs-server.service - NFS server and services.
```

- Edit the /etc/exports config file that specifies the path to the NFS volume you'd like to share. in my example, I used:

```/media/vault/```

- vim / nano : ```/etc/exports``` and add the following line:

```
  # /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#

/media/vault 192.168.2.0/24(rw,sync,no_subtree_check)
  
```

 - The above config just specifies the directories we're sharing and who can access them. I used 192.168.2.0/24 for a range of IP addresses that can connected to this NFS server. You may also specify just one IP. the options for the share are:
    
      * rw = read/write. Read/Write or Read-Only access for clients.
      * sync = changes to the files in the folder are sychronized immediately. Server responds before disk write (faster, less reliable).
      * no_subtree_check =  when a directory is shared of a network, the system checks each subfolder within the exported; shared folder for additional access permissions. This disables subtree checking for performance (use with caution).
   
- Once the configs are set, run ```exportfs -ar```. look out for any errors in the terminal. anytime you make changes to the /etc/exports config file, you'll need to run ```exportfs -ar```.


- to view the exports that were created, just run ```exportfs -v```. Example:

```
root@storageNode:/media/vault# exportfs -v 
/media/vault    192.168.2.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
root@storageNode:/media/vault# 
```

- Allow ingress traffic to port 2049 for NFS :

```
ufw allow from 192.168.2.0/24 to any port 2049
```

**client side setup will be next.**
