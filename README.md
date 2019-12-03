### Preparation
1. [Connect Virtual Machines](https://github.com/Artlands/Install-Slurm/tree/master/Connect_VM)

2. [Setup NFS Server](https://github.com/Artlands/Install-Slurm/tree/master/Setup_NFS)

### Cluster Server and Computing Nodes
List of master node and computing nodes within the cluster.

|Hostname|IP Addr |
|--------|--------|
|master  |10.0.1.5|
|node1   |10.0.1.6|
|node2   |10.0.1.7|

### (Optional) Delete failed installation of Slurm

Remove database.

```
yum remove mariadb-server mariadb-devel -y
```

Remove Slurm and Munge.

```
yum remove slurm munge munge-libs munge-devel -y
```

Delete the users and corresponding folders.

```
userdel -r slurm
suerdel -r munge
```

### Create the global users

Slurm and Munge require consistent UID and GID across every node in the cluster. For all the nodes, before you install Slurm or Munge:

```
export MUNGEUSER=991
groupadd -g $MUNGEUSER munge
useradd  -m -c "MUNGE Uid 'N' Gid Emporium" -d /var/lib/munge -u $MUNGEUSER -g munge  -s /sbin/nologin munge
export SLURMUSER=992
groupadd -g $SLURMUSER slurm
useradd  -m -c "SLURM workload manager" -d /var/lib/slurm -u $SLURMUSER -g slurm  -s /bin/bash slurm
```

### Install Munge

Get the latest REPL repository.

```
yum install epel-release -y
```

Install Munge.

```
yum install munge munge-libs munge-devel -y
```

Create a secret key on __master__ node. First install rig-tools to properly create the key.

```
yum install rng-tools -y
rngd -r /dev/urandom
/usr/sbin/create-munge-key -r
dd if=/dev/urandom bs=1 count=1024 > /etc/munge/munge.key
chown munge: /etc/munge/munge.key
chmod 400 /etc/munge/munge.key
```

Send this key to all of the compute nodes.

```
scp /etc/munge/munge.key root@10.0.1.6:/etc/munge
scp /etc/munge/munge.key root@10.0.1.7:/etc/munge
```

SSH into every node and correct the permissions as well as start the Munge service.

```
chown -R munge: /etc/munge/ /var/log/munge/
chmod 0700 /etc/munge/ /var/log/munge/
```

```
systemctl enable munge
systemctl start munge
```

To test Munge, try to access another node with Munge from __master__ node.

```
munge -n
munge -n | munge
munge -n | ssh 10.0.1.6 unmunge
remunge
```

If you encounter no errors, then Munge is working as expected.

### Install Slurm

Install a few dependencies.

```
yum install openssl openssl-devel pam-devel numactl numactl-devel hwloc hwloc-devel lua lua-devel readline-devel rrdtool-devel ncurses-devel man2html libibmad libibumad -y
```

Download the latest version of Slurm in the shared folder.

```
cd /nfsshare
wget https://download.schedmd.com/slurm/slurm-19.05.4.tar.bz2
```

If you don't have `rpmbuild` yet.

```
yum install rpm-build
rpmbuild -ta slurm-19.05.4.tar.bz2
```

Check the rpms created by `rpmbuild`.

```
cd /root/rpmbuild/RPMS/x86_64
```

Move the Slurm rpms for installation for all nodes.

```
mkdir /nfsshare/slurm-rpms
cp * /nfsshare/slurm-rpms
```

On every node, install these rpms.

```
yum --nogpgcheck localinstall * -y
```

Reference: [slothparadise.com](https://www.slothparadise.com/how-to-install-slurm-on-centos-7-cluster/)