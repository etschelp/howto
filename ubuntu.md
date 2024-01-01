# Ubuntu Administration

## Setup

### Add non root user

```sh
adduser <username>
usermod -aG sudo <username>
```

### Optional: Public/Private Key

```sh
ssh-keygen -t rsa -b 4096
ssh-copy-id <username>@<host>
```

### SSH Config

`sudo vi /etc/ssh/sshd_config`

To stop accepting locale on the server uncomment:

`AcceptEnv LANG LC_*`

#### Disable Password Authentication

```sh
PasswordAuthentication no
PermitRootLogin no
```

`sudo service sshd restart`

### Configure Timezone

`sudo dpkg-reconfigure tzdata`

### Configure package management

`sudo dpkg-reconfigure unattended-upgrades`

The edit the file:

`/etc/apt/apt.conf.d/50unattended-upgrades`

To enable completely automatic autoremove set:

`Unattended-Upgrade::Remove-Unused-Dependencies "true";`

### Check Swap File Size

```sh
sudo swapon -s
sudo swapoff /swapfile
sudo fallocate -l 4G /swapfile
sudo mkswap /swapfile
sudo chmod 0600 /swapfile
sudo swapon /swapfile
sudo swapon -s
```

### Set Swappiness To 0

Get current value:

`cat /proc/sys/vm/swappiness` 

Set new value:

`sudo vi /etc/sysctl.conf` -> `vm.swappiness = 0`

This needs a reboot to be effective. To do this without reboot:

`sysctl vm.swappiness=0`
