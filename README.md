# Installation guide for Plex Media Server on Turris Omnia

> Installation steps for OOBE LXC Turris Omnia Debian container.

## Overview

The installation consists of these steps:

1. Create Debian LXC container
2. Simple Debian configuration
3. Install Plex Media Server
4. Connect to CIFS

## 1. Create Debian LXC container

Connect to your Turrins Omnia router by SSH and create the LXC container. **[Official Manual](https://www.turris.cz/doc/en/howto/lxc)**
```
lxc-create -t download -n PLEX
```

- Distribution: **Debian**
- Release: **Jessie**
- Architecture: **armv7l**

## 2. Simple Debian configuration


Start the container from LuCI or by executing this command:

```
lxc-start -n PLEX
```

Connect to container:

```
lxc-attach -n PLEX
```

Install prerequisites:

```
apt-get update
apt-get install sudo
apt-get install nano
```

Change hostname (for eg. `PLEX`):

```
nano /etc/hostname
```

Set new hostname to localhost:

```
nano /etc/hosts
```


Add this line (for `PLEX` as hostname):

```
127.0.1.1   PLEX
```

Install packages:

```
apt-get install git-core openssh-server rsync sudo fakeroot cifs-utils -y
```

Reboot the LXC container from LuCI interface or execute this command:

```
lxc-stop -n PLEX -r
```

Following steps are not really necessary and you can continue to No. 3 if you don't need a direct access to the container. Actually, direct access only complicates mounting process later on.

Create your user (eg. `johndoe`):

```
adduser johndoe
```

Run `visudo` commnad and add:

```
johndoe ALL=(ALL:ALL) ALL
```

Log to your user using SSH or sudo:

```
sudo su johndoe
```

## 3. Install Plex Media Server

Add repository

```
echo "deb http://dl.bintray.com/tproenca/pmsarm7 jessie main" | tee /etc/apt/sources.list.d/pms.list
```

or 

```
echo "deb http://dl.bintray.com/tproenca/pmsarm7 jessie main" | sudo tee /etc/apt/sources.list.d/pms.list
```
if you are under different account. Same analogy is used with other commands.


Fix for automatic start:

```
touch /usr/lib/plexmediaserver/start.sh
```

or

```
sudo touch /usr/lib/plexmediaserver/start.sh
```

Install:

```
apt-get update
apt-get install plexmediaserver
```

or

```
sudo apt-get update
sudo apt-get install plexmediaserver
```

## 4. Connect to CIFS

Write down `plex` user id (`uid`):

```
id -u plex
```

Create empty folder for mount:

```
mkdir -p /media/plex
```

Add mount to `/etc/fstab` (set `uid` from `plex` user)

```
//192.168.1.1/nas /media/plex cifs uid=107,gid=1000,iocharset=utf8 0 0
```
(replace with appropiate paths)

Now, if you are logon with other user than root, you have to detach from that user and attach to the PLEX host containter again. That will give you an access under root account to execute mount.

Execute following command:
```
mount -all
```

If you are not able to mount or you want an automatic mount without any password prompt, try execute this:
```
nano /etc/.cifspasswd
```

Add your credentials to the file:
```
username=SAMBA_USER_NAME
password=SAMBA_USER_PSWD
```

Alternativelly, you can create the `/etc/.cifspasswd` file by WinSCP.

Replace the previous `/etc/fstab` entry to this:
```
//192.168.1.1/nas /media/plex cifs uid=107,gid=1000,credentials=/etc/.cifspasswd,iocharset=utf8 0 0
```

## 5. PLEX server configuration

You can check the PLEX server status by:
```
systemctl status plexmediaserver
```

If Plex media server isn’t running, you can start it with:
```
systemctl start plexmediaserver
```

or

```
sudo systemctl start plexmediaserver
```

Once the PLEX server is running, visit the webpage http://192.168.1.100:32400/manage (substitude the IP with the IP of the LXC container) and create the libraries.

Congratulation, you have your PLEX media server up and running on Turris Omnia device.

## Links

[For more about CIFS mount](http://midactstech.blogspot.cz/2013/09/how-to-mount-windows-cifs-share-on_18.html)<br>
[How to Install Plex Media Server on Ubuntu](https://www.linuxbabe.com/ubuntu/install-plex-media-server-ubuntu-18-04)
## Credits

[Plex Media Server ARM package for Debian/Ubuntu Linux](https://tproenca.github.io/pmsarm7/)
