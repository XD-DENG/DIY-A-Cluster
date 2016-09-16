
## Network Basics

- [How to Set Static IP Address](#how-to-set-static-ip-address)
- [How to Set Password-less Access](#how-to-set-password-less-access)
- [How to Copy Files & Folders From One Node to Another](#how-to-copy-files--folders-from-one-node-to-another)
- [How to Disable Wi-Fi Interface](#how-to-disable-wi-fi-interface)

<br><br>

### How to Set Static IP Address

The way to set IP address manually may be a bit different. Here I would introduce how-to-do on 

- Raspberry Pi
- Debian (or Ubuntu)

#### Raspberry Pi
We can set static IP address by modifying file `/etc/dhcpcd.conf`. Append the contents below to file `/etc/dhcpcd.conf`

```{bash}
interface eth0

static ip_address=172.24.210.100/24
static routers=172.24.210.1
static domain_name_servers=172.24.210.1
```

**interface**: This defines which network interface you are setting the configuration for. If you're using wired connection, the interface should be `eth0`. It should be `wlan0` if you're using wireless connection.

**static ip_address** = This is the IP address that you want to set your device to. (Make sure you leave the /24 at the end)

**static routers** = This is the IP address of your gateway (probably the IP address or your router)

**static domain_name_servers** = This is the IP address of your DNS (probably the IP address of your router). You can add multiple IP addresses here separated with a single space.

Then we need to reboot the machine and we can use comman `ifconfig` to check if the setting is successful.

For my cluster, I set the IP of master node to be `172.24.210.100`, and the IP addresses of two worker nodes to be `172.24.210.101` and `172.24.210.102`.


#### Debian (or Ubuntu)

The majority of network setup on Debian platform can be done via the `interfaces` configuration file at `/etc/network/interfaces`. You can give your network card an IP address, set up routing information, configure IP masquerading, set default routes and much more [9].

To set a static IP address, you can add rows like below into `/etc/network/interfaces` (network, broadcast and gateway are optional)

```{bash}
auto eth0
iface eth0 inet static
    address 172.24.210.101
    netmask 255.255.255.0
    gateway 172.24.210.1
```


<br><br>


### How to Set Password-less Access

When we set Hadoop and Spark clusters, the master machine accesses each of the worker (slave) machines via **ssh**. Pawword-less (using a private key) access is required then.

First, we need to generate the private key on the master machine

```{bash}
ssh-keygen -t rsa -P ''
```

the argument `-P ''` is to tell the command that the password is empty.

Then we can use the command below to copy the key to the worker machine

```
ssh-copy-id slave_user_ID@slave_node_IP
```

After this, you can check file `~/.ssh/authorized_keys` on slave machine to check if the key is passed to the slave machine. You can also run `ssh slave_user_ID@slave_node_IP` on master machine to check whether password-less access is set up successfully.


<br><br>


### How to Copy Files & Folders From One Node to Another

Since we need to make sure our nodes are always on the same page, instead of doing the same thing repeatedly on a few nodes, we can do it once and then copy relevant files and folders to another node. To do this, we can use the `scp` (Secure Copy) command in Linux. `scp` allows files to be copied to, from, or between Linux machines. It uses **ssh** for data transfer and provides the same level of security as **ssh** [6].

A few example syntax are listed below [6]:

Copy the file "foobar.txt" from the local host to a remote host

```
$ scp foobar.txt your_username@remotehost.edu:/some/remote/directory
```

Copy the file "foobar.txt" from a remote host to the local host

```
$ scp your_username@remotehost.edu:foobar.txt /some/local/directory
```

Copy the directory "foo" from the local host to a remote host's directory "bar"

```
$ scp -r foo your_username@remotehost.edu:/some/remote/directory/bar
```

Copy the file "foobar.txt" from remote host "rh1.edu" to remote host "rh2.edu"

```
$ scp your_username@rh1.edu:/some/remote/directory/foobar.txt \
your_username@rh2.edu:/some/remote/directory/
```

Copying the files "foo.txt" and "bar.txt" from the local host to your home directory on the remote host

```
$ scp foo.txt bar.txt your_username@remotehost.edu:~
```

Copy the file "foobar.txt" from the local host to a remote host using port 2264

```
$ scp -P 2264 foobar.txt your_username@remotehost.edu:/some/remote/directory
```

Copy multiple files from the remote host to your current directory on the local host

```
$ scp your_username@remotehost.edu:/some/remote/directory/\{a,b,c\} .
$ scp your_username@remotehost.edu:~/\{foo.txt,bar.txt\} .
```


<br><br>


### How to Disable Wi-Fi Interface

This is useful when you're using a mobile device as one of your nodes, like Raspberry Pi 3 (for a laptop, you can use some GUI tool to disable the Wi-Fi module). To build a cluster, it's much more efficient to used wired connections among nodes than using wireless connection, like Wi-Fi. Then we may need to disable the Wi-Fi interface, which is typically `wlan0`.

To disable this interface, it's quite simple [3]

```
sudo ifdown wlan0
```

If you want to get it back later, run

```
sudo ifup wlan0
```









