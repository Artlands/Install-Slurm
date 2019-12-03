### Prequisites
1. [VirtualBox](virtualbox.org/wiki/Downloads)
2. [CentOS 7.7](http://repo1.dal.innoscale.net/centos/7.7.1908/isos/x86_64/) (minimal version)

### Create virtual machines in VirtualBox
Set Network Adapter 1 to `NAT`, Adapter 2 to `Internal Network`

### Setting up Internet Connecttion and DHCP
1. Create an Inernal Network with VirtualBox's built-in command. It will mimic DHCP server.

In terminal, type:
``` 
VBoxManage dhcpserver add --netname intnet --ip 10.0.1.1 --netmask 255.255.255.0 --lowerip 10.0.1.2 --upperip 10.0.1.200 --enable
```

2. Setup the internet connection on each virtual machine:

``` 
nmcli d
```

Show "enp0s3", "enp0s8". "enp0s8" will be connected, "enp0s3" will be disconnected:

``` 
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
```

Change ONBOOT=yes:
``` 
ONBOOT=yes
```
Restart network:
``` 
systemctl restart network
```

Show IP addresses:
``` 
ip addr show
```

Try to communicate within the internal network among machines

On master node: `ping 10.0.1.6`, `ping 10.0.1.7`
On computing node[1-2]: `ping 10.0.1.5`

We have established the ability to ping from one machine to the other machine.

### Setting up SSH Keys

On __master__, create the ~/.ssh folder:

``` 
mkdir ~/.ssh
```

```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

You can press `Enter` to leave the next three propmts as default.

```
Enter file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]
Enter passphrase (empty for no passphrase): [Type a passphrase]
Enter same passphrase again: [Type passphrase again]
```

We will copy the public key `id_rsa.pub` to `authorized_keys` to enable this key for access to __master__:

```
cp id_rsa.pub authorized_keys
```

Send the private key `id_rsa` and public key `id_rsa.pub` from __master__ to __node[1-2]__:

```
scp ~/.ssh/id_rsa ~/.ssh/id_rsa.pub root@10.0.1.6:
scp ~/.ssh/id_rsa ~/.ssh/id_rsa.pub root@10.0.1.7:
```

Make the ~/.ssh directory on __node[1-2]__:

```
mkdir ~/.ssh
```

Copy the `id_rsa` and `id_rsa.pub` to the ~/.ssh folder:

```
cp id_rsa id_rsa.pub ~/.ssh
```

Copy `id_rsa.pub` to the `authorized_kyes` to allow __master__ to be able to SSH to __node[1-2]__ without a password:

```
cd ~/.ssh
cp id_rsa.pub authorized_keys
```

We should be able to ssh form __master__ to __node[1-2]__ without a password and vice versa.

Reference: [slothparadise.com](https://www.slothparadise.com/how-to-connect-virtual-machines-and-setup-nfs-server-part-1/)

