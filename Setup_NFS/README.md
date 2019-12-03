### Setting up NFS server: master

Install a couple of NFS libraries:

```
yum install nfs-utils nfs-utils-lib -y
```

```
systemctl enable rpcbind nfs-server
systemctl start rpcbind nfs-server
```

Create a new empty folder that will be the shared folder:

```
mkdir /nfsshare
```

```
vi /etc/exports
```

Add the following lines to /etc/exports. We write the name of the shared folder and the IP addresses that we want the shared folder to be shared:

```
/nfsshare 10.0.1.6(rw,sync,no_root_squash,no_subtree_check)
/nfsshare 10.0.1.7(rw,sync,no_root_squash,no_subtree_check)
```

Load the /etc/exports new changes:

```
exportfs -a
```

Change the firewall to allow NFS and complimenting services:

```
firewall-cmd --permanent --zone=public --add-service=nfs
firewall-cmd --permanent --zone=public --add-service=mountd
firewall-cmd --permanent --zone=public --add-service=rpc-bind
firewall-cmd --reload
```

Restart NFS service:

```
systemctl restart nfs
```

### Setting up NFS Client: node1, node2
Install a couple of NFS libraries:

```
yum install nfs-utils nfs-utils-lib -y
```

Make a folder where the shared folder from __master__ will be mounted on __node1__, __node2__:

```
mkdir /nfsshare
```

Make sure that the following two commands do not return any errors:

```
showmount -e 10.0.1.5
rpcinfo -p 10.0.1.5
```

```
mount 10.0.1.5:/nfsshare /nfsshare
```

To see if  10.0.1.5:/nfsshare mount has been created:

```
df -h
```

Test the shared folder actually works:

```
touch /nfsshare/test.txt
```

On __master__, if we cd /nfsshare, we will see test.txt is inside the folder.

### Making NFS automatic

Set autoatic way for the NFS client to look for the NFS folder. On __node[1-2]__:

```
vi /etc/fstab
```

Add the following line:

```
10.0.1.5:/nfsshare /nfsshare nfs auto,noatime,nolock,bg,nfsvers=3,intr,tcp,actimeo=1800 0 0
```

Every time we restart the __node[1-2]__, we can re-mount the NFS shared folder by typing:

```
mount -a
```

Reference: [slothparadise](https://www.slothparadise.com/how-to-connect-virtual-machines-and-setup-nfs-server-part-1/)

