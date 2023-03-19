# PiVPN
Runs on LXC container running Ubuntu 18.04 on pve

## Installing PiVPN

1. Install a LXC container with Ubuntu 18.04. I am using 2GB memory and 1GB swap

2. Expose Container

```
cd /etc/pve/lxc
nano (container number)
add these lines
lxc.cgroup.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net dev/net none bind,create=dir
run this command
chown 100000:100000 /dev/net/tun
check that it worked
ls -l /dev/net/tun
output should be something like this
crw-rw-rw- 1 100000 100000 10, 200 Dec 22 13:26 /dev/net/tun
```

3. Installing WireGuard:

```
sudo apt update && sudo apt upgrade -y 
sudo apt install curl -y
https://www.pivpn.io/
curl -L https://install.pivpn.io | bash
```

4. Add environment variable to run ´pivpn´ commands
```
echo "export PATH="/usr/local/bin:$PATH" >> .bashrc
source .bashrc
```
