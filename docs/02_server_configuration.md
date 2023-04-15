# Server configuration

Note: This document is part of the documentation "[Howto build a website on wordpress](../README.md)".
You might want to start from the very beginning of it.

## Basic server configuration

I open a shell / terminal / console on my (I also use Linux) PC and connect to the new server via ssh:

```shell
ssh root@<my-server-IPv4-address>       # command to login server through ssh
# acknowledge question about server fingerprint
# the root password will be asked for login
```

### Update & install helpful tools

Then I update the operating system (not knowing how old the hosting providers images are):

```shell
apt update && apt upgrade -y    # update package cache and install updates
apt autoremove --purge          # uninstall dependencies that are not required anymore
```

For the next steps it will be nice to have some tools installed:

```shell
apt install nano        # terminal editor to edit files
apt install tmux        # terminal emulator to split window into multiple shells
apt install htop        # system ressource viewer like RAM and processes
apt install net-tools   # network tools to check open ports
```

### Add another user

Working as root on a server I always feel uncomfortable. The user name is known by any attacker. That's why I add a new user:

```shell
adduser johndoe             # generate user and home folder /home/johndoe
# a password for the new user will be asked twice

usermod -aG sudo johndoe    # allow "sudo" for the new user
```

From my home PC I upload my ssh keys for the new user:

```shell
ssh-copy-id -i /path/to/my/ssh-key johndoe@<my-server-IPv4-address> # add ssh key for new user on server
ssh johndoe@<my-server-IP>                                          # login with new user on server
# now ssh key passphrase (instead of password) will be asked
```                                         

and on the server I try "sudo" with new user:

```shell
sudo su     # become "root" without any error output
```

### Configure ssh

Now I can disable ssh access for user root:

```shell
sudo nano /etc/ssh/sshd_config
# replace PermitRootLogin yes with no
# CTRL+o to save and CTRL+x to end
# takes effect after reboot (I'll do later)
```

I also run my ssh server on a custom port. In my /var/log/auth.log I discovered bots stop trying to login after that change:

```shell
sudo nano /etc/ssh/sshd_config
# replace "Port 22" with Your custom port
# CTRL+o to save and CTRL+x to end
# takes effect after reboot (I'll do later)
```

### Add swap

A servers memory / RAM is limited. As I mentioned earlier my server only has 2GB. If any service would consume more memory, my server would crash and needed to be rebooted. That's why I add "swap", as a kind of additional memory buffer for emergency cases. Later I check swap consumption (with htop) from time to time (what is always the case) and see if I need to have more memory in general (what is only the case is memory is always full).

```shell
sudo su
touch /var/swap.1                                           # create a swap file
dd if=/dev/zero of=/var/swap.1 bs=1MiB count=$((1*5120))    # fill file with 5GB zeros
chmod 600 /var/swap.1                                       # only root may handle memory
mkswap /var/swap.1                                          # make swap filesystem
swapon /var/swap.1                                          # activate swap for now
echo "/var/swap.1 none swap defaults 0 0" >> /etc/fstab     # activate swap after reboot
swapon --show                                               # will show /var/swap.1 file   5G 0M   -2
htop                                                        # will show Swp[|               0M/5.00G]
```

### set timezone

To avoid fighting against wrong timestamps in different services, it is really helpful to set the servers time zone properly and (through docker-compose.yml) inject this into all docker containers. To check and (if required) set time zone on the server these commands are helpful:

```shell
# check servers current time
date
# check if current server timezone is set to Your location
timedatectl
# check availabe timezones
timedatectl list-timezones
# run process to set another time zone
sudo dpkg-reconfigure tzdata
```

## Reboot and check

After these changes it makes sense to reboot:

```shell
sudo reboot
```

and see if:

- the machine gets up and running again (You can login with ssh)
- ssh will be available at the new port

```shell
ssh johndoe@<my-servers-IPv4-address> -p <my-custom-port>  # should work
```

- check if ssh login as root is blocked

```shell
ssh root@<my-servers-IPv4-address> -p <my-custom-port>     # should not work
```

- the swap is again enabled (swapon --show or htop)
- the volume (40GB that I booked at webhoster) is mounted

```shell
cat /etc/fstab | grep -i volume   # non-empty output like: /dev/disk/by-id/scsi-0HC_Volume_1234567 ...
mount | grep -i volume            # non-empty output like: /dev/sdb on /mnt/volume-fsn1-1 type...
```
Notice: You see my volume is mounted to /mnt/volume-fsn1-1. That is important, because all my future work will be done here!

- no unexpected network ports on my system are open:

```shell
sudo netstat -tulpen
# Active Internet connections (only servers)
# Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode      PID/Program name    
# tcp        0      0 0.0.0.0:12345           0.0.0.0:*               LISTEN      0          17450      641/sshd: /usr/sbin
# tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      102        16729      546/systemd-resolve
# tcp6       0      0 :::12345                :::*                    LISTEN      0          17466      641/sshd: /usr/sbin 
# udp        0      0 127.0.0.53:53           0.0.0.0:*                           102        16728      546/systemd-resolve 
# udp        0      0 my.ext.IPv4.123:68      0.0.0.0:*                           101        6990286    544/systemd-network
```
Notice: You see only "internal stuff" and my custom ssh port 12345 are open

- the time and time zone is set well (command: date)


## Install docker & docker-compose

Through the internet are many descriptions, how docker can be installed from dockers official software repository. Since I work with docker from Ubuntu repos I never had problems and so install it the easy way:

```shell
sudo apt install docker.io docker-compose
```

Docker will later download images and create containers for me. I want these objects to be stored on my 40GB volume to be automatically migrated to another server when I mount the volume there. That's why I tell docker, to no use its default storage place, but store data on my volume:

```shell
sudo su
mkdir -p /mnt/volume-fsn1-1/docker_data
nano /etc/docker/daemon.json
# I added this content (without #):
# {
#    "data-root": "/mnt/volume-fsn1-1/docker_data"
# }
systemctl stop docker                                           # stop all docker stuff
rsync -axPS /var/lib/docker/ /mnt/volume-fsn1-1/docker_data     # mv existing data to new folder
systemctl daemon-reload                                         # make config file known
systemctl start docker                                          # start docker again
docker info | grep "Root Dir"                                   # should show Your root dir now
```

For detailled help see [this article](https://stackoverflow.com/questions/55344896/attempt-to-change-docker-data-root-fails-why).


Well, thats the basic server configuration. See [next chapter](./03_webserver_configuration.md) how I configured webserver related stuff.
