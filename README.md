# Installation guide for Plex Media Server on Turris Omnia

> Installation steps for OOBE LXC Turris Omnia Debian container.

## Overview

The installation consists of these steps:

1. Configure NAS
2. Create Debian LXC container
3. Simple Debian configuration
4. Install Plex Media Server
5. Connect to CIFS

## 1. Configure NAS

Follow [the official tutorial](https://wiki.turris.cz/doc/en/howto/nas) to configure NAS at your Turris Omnia. This tutorial is using NAS path configuration as `\\TURRIS\nas` or using IP form `\\192.168.1.1\nas`.

## 2. Create Debian LXC container

Connect to your Turris Omnia router by SSH and create the LXC container. **[Official Manual](https://www.turris.cz/doc/en/howto/lxc)**

```bash
lxc-create -t download -n PLEX
```

- Distribution: **Debian**
- Release: **Stretch**
- Architecture: **armv7l** (ending 7l is seven with the lower case L, NOT seventy-one)

## 3. Simple Debian configuration

Start the container from LuCI or by executing this command:

```bash
lxc-start -n PLEX
```

Connect to container from a console:

```bash
lxc-attach -n PLEX
```

Install prerequisites:

```bash
apt-get update
apt-get install sudo nano wget git-core openssh-server rsync sudo fakeroot cifs-utils -y
```

Change hostname (for eg. `PLEX`):

```bash
nano /etc/hostname
```

Save file by hitting CTRL+X in nano editor.

Set new hostname to localhost:

```bash
nano /etc/hosts
```

Add this line (for `PLEX` as hostname):

```bash
127.0.1.1   PLEX
```

Exit the attached container simply by typing `exit`. For the next step, you should be your Turris as admin in your console.

Reboot the LXC container from LuCI interface or execute this command:

```bash
lxc-stop -n PLEX -r
```

## 4. Install Plex Media Server

Connect to container again. From a console attached to Turris, you can execute command:

```bash
lxc-attach -n PLEX
```

Go to [PLEX Server download page](https://www.plex.tv/media-server-downloads/) and copy the latest installation package for Ubuntu/Debian (8+) ARMv7 package or just copy/paste the command below (_the latest version in time of writting this commit_).

![Preview](https://raw.githubusercontent.com/KUTlime/Installation-guide-for-Plex-Media-Server-on-Turris-Omnia/master/OfficialRepo.png)

```bash
wget https://downloads.plex.tv/plex-media-server-new/1.21.3.4046-3c1c83ba4/debian/plexmediaserver_1.21.3.4046-3c1c83ba4_armhf.deb
```

Now, install the downloaded package. If you've downloaded some other package, replace the package name accordingly.

```bash
dpkg -i plexmediaserver_1.21.3.4046-3c1c83ba4_armhf.deb
```

Fix for automatic start:

```bash
touch /usr/lib/plexmediaserver/start.sh
```

## 5. Connect to CIFS

Write down `plex` user id (`uid`):

```bash
id -u plex
```

Create empty folder for mount:

```bash
mkdir -p /media/plex
```

Add mount to `/etc/fstab` by editing file in the `nano` editor:

```bash
nano /etc/fstab
```

and write the following line (_set `uid` from `plex` user and replace paths with appropriate paths from your environment_):

```bash
//192.168.1.1/nas /media/plex cifs uid=999,gid=1000,credentials=/etc/.cifspasswd,iocharset=utf8 0 0
```

For Turris Omnia OS 5.1.x with Samba4, use this command:

```bash
//192.168.1.1/nas /media/plex cifs uid=999,vers=2.0,gid=1000,credentials=/etc/.cifspasswd,iocharset=utf8 0 0
```

For an automatic mount without any password prompt, try execute this:

```bash
nano /etc/.cifspasswd
```

Add write your credentials to the file:

```bash
username=SAMBA_USER_NAME
password=SAMBA_USER_PSWD
```

Mostly, you would go with:

```bash
username=root
password=YOUR_TURRIS_ROOT_PASSWORD
```

Save and exit the `/etc/.cifspasswd` file.

Alternatively, you can create the `/etc/.cifspasswd` file by WinSCP.

Execute following command:

```bash
mount -all
```

Verify the mount by:

```bash
cd /media/plex  && ls
```

You should see your files or/and directories for PLEX.

## 6. PLEX server configuration

Find IP address of your LXC container and write down this IP:

```bash
ip a
```

The output is messy, so look for a valid LAN IP with `inet` at the beginning.<br>
Alternatively, you can find the IP in the your DHCP in LuCI.

You can check the PLEX server status by:

```bash
systemctl status plexmediaserver
```

If PLEX media server is running, close the console. We are done here.<br>
Otherwise, you can start the service with:

```bash
systemctl start plexmediaserver
```

Once the PLEX server is running, visit the webpage http://192.168.1.100:32400/manage (*substitute the IP with the IP of the LXC container*) and create the libraries. If this webpage is doesn't work, try http://192.168.1.100:32400/web/index.html.

Congratulation, you have your PLEX media server up and running on Turris Omnia device.

## Enable repository updating

To enable an easy update of Plex media server, you have to add Plex official repository to your container. Start with installing prerequisites with:

```bash
apt-get install curl apt-transport-https
```

After that, execute:
```bash
echo deb https://downloads.plex.tv/repo/deb public main | sudo tee /etc/apt/sources.list.d/plexmediaserver.list
```

and finish the setup with:

```bash
curl https://downloads.plex.tv/plex-keys/PlexSign.key | sudo apt-key add -
```

After, you will be able to update your Plex media server just by running

```bash
apt-get update
```

## Automated updates via cron

Create a shell script with following commands:

```bash
export PATH="$PATH:/usr/local/sbin:/usr/sbin:/sbin"
apt-get update
apt-get upgrade -y
apt-get autoclean
```

Place it to some location, e.g., `/root/installUpdate.sh`.

In terminal, execute these commands:

```bash
apt-get install cron -y
chmod +x /root/installUpdate.sh
crontab -e
```

Insert a following line of code:

```bash
44 4 * * * /root/installUpdate.sh >> /var/log/plexupdatejob.log 2>&1
```

This will execute the script every single day at 04:44.

## Links & Credits

[READ ME FIRST: About Server ARMv7 and ARMv8 Ubuntu / Debian](https://forums.plex.tv/t/read-me-first-about-server-armv7-and-armv8-ubuntu-debian/226567)  
[PLEX Linux installation](https://support.plex.tv/articles/200288586-installation/)  
[For more about CIFS mount](http://midactstech.blogspot.cz/2013/09/how-to-mount-windows-cifs-share-on_18.html)  
[How to Install Plex Media Server on Ubuntu](https://www.linuxbabe.com/ubuntu/install-plex-media-server-ubuntu-18-04)  
[Enable repository updating for supported Linux server distributions](https://support.plex.tv/articles/235974187-enable-repository-updating-for-supported-linux-server-distributions/)
