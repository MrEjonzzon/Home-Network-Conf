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
Choose DHCP during installation wizard, otherwise use default.

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

## Set up port
I am using an EdgeRouter, should be similar on other devices.
1. Go to https://192.168.1.1
2. Firewall/NAT > Port Forwarding
![image](https://user-images.githubusercontent.com/70457382/226215795-5efd23c4-cb7c-45b0-beee-649f14a419ad.png)

## Add client
```
pivpn -a
add conf file to the client
```

### Windows/MacOS
1. Install WireGuard client
2. Load the conf file 

### Linux/Debian
```
sudo add-apt-repository ppa:wireguard/wireguard
sudo apt install wireguard
sudo apt install resolvconf
cd /etc/wireguard
vim wg0.conf > add content of conf file on PiVPN server
systemctl start wg-quick@wg0
systemctl enable wg-quick@wg0
```

I have been using the following guides:

https://wiki.homelabtim.com/books/easiest-vpn-on-proxmox-with-wireguard/page/commands-used-in-video

https://www.vpnunlimited.com/help/manuals/wireguard/linux

https://www.crosstalksolutions.com/pivpn-wireguard-complete-setup-2022/
