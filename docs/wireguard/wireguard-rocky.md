---
title: Assign Public IPv4 & IPv6 Wireguard Rocky Linux
sidebar_position: 1
---

# Assign Public IPv4 & IPv6 Wireguard Rocky Linux
Tested in Rocky Linux 9

## Prerequisite
1. IPv4 allocation minimum /30 or IPv6
2. Client Server
3. VPS or VM for Wireguard Server
    - SELinux is running in permissive mode
```bash
setenforce permissive
reboot
```
    - Disable Firewalld
```bash
systemctl stop firewalld
systemctl disable firewalld
```

## Enable Wireguard Kernel Module
```bash
modprobe wireguard
lsmod | grep wireguard
```

```bash
echo wireguard > /etc/modules-load.d/wireguard.conf
```

## Install Wireguard
```bash
dnf install wireguard-tools
```

## Generate Server Key Pair
```bash
wg genkey | tee /etc/wireguard/server.key
chmod 0400 /etc/wireguard/server.key
```
```bash
cat /etc/wireguard/server.key | wg pubkey | tee /etc/wireguard/server.pub
```
```bash
cat /etc/wireguard/server.key
cat /etc/wireguard/server.pub
```

## Generate Client Key Pair
```bash
mkdir -p /etc/wireguard/clients
```
```bash
wg genkey | tee /etc/wireguard/clients/client1.key
cat /etc/wireguard/clients/client1.key | wg pubkey | tee /etc/wireguard/clients/client1.pub
```
```bash
cat /etc/wireguard/clients/client1.key
cat /etc/wireguard/clients/client1.pub
```

## Configure Wireguard - Server Side
```bash
vi /etc/wireguard/wg0.conf
```
### Server Side Configuration
```bash
[Interface]
# Wireguard Server private key - server.key
PrivateKey = # Copy Server private key here
# Wireguard interface will be run at 10.8.0.1
Address = 10.8.0.1/24, fd00::1/64 #any IP private network

# Clients will connect to UDP port 51820
ListenPort = 51820

# Ensure any changes will be saved to the Wireguard config file
SaveConfig = true

# Change IPv6_CLIENT_ASSIGN to ipv6 public and ens33 to your interface
PostUp=ip -6 neigh add proxy IPv6_CLIENT_ASSIGN dev ens33
PostDown=ip -6 neigh del proxy IPv6_CLIENT_ASSIGN dev ens33

[Peer]
# Wireguard client public key - client1.pub
PublicKey = # Copy client public key here
# clients' VPN IP addresses you allow to connect
# possible to specify subnet ⇒ [172.16.100.0/24]
AllowedIPs = # copy IP Public/cidr
```
### Port Forwarding
```bash
vi /etc/sysctl.conf
```
```bash
# Port Forwarding for IPv4
net.ipv4.ip_forward=1
net.ipv4.conf.all.proxy_arp=1

# Port forwarding for IPv6
net.ipv6.conf.all.forwarding=1
net.ipv6.conf.all.proxy_ndp=1
```

```bash
sysctl -p
reboot #prefered
```

### Start Wireguard Server
```bash
systemctl start wg-quick@wg0.service
systemctl enable wg-quick@wg0.service
systemctl status wg-quick@wg0.service

```


## Configure Wireguard - Client Side
This configuration is implemented on your client-side. Adjust based on your system.
```bash
vi /etc/wireguard/wg0.conf
```
### Client Side Configuration
```bash
[Interface]
Address = # copy ip public allocation for client
# Wireguard Client private key - client1.key
PrivateKey = # Copy client private key here

[Peer]
# Wireguard Server public key - server.pub
PublicKey = #Copy server public key here
AllowedIPs = 0.0.0.0/0,::/0 # makes your home server send all outbound packets via this tunnel
Endpoint = # copy wireguard ip public:port
# Sending Keepalive every 25 sec
PersistentKeepalive = 25
```

## Maintainance Server
any change in `wg0.conf` need stop the wg first, update the conf, and start again.
```bash
wg-quick down /etc/wireguard/wg0.conf
wg-quick up /etc/wireguard/wg0.conf
```

## Reference
- [How to Install Wireguard VPN on Rocky Linux 9](https://www.howtoforge.com/how-to-install-wireguard-vpn-on-rocky-linux-9/)
- [How to assign public IPv4 to a Wireguard client?](https://www.reddit.com/r/WireGuard/comments/ld09tr/how_to_assign_public_ipv4_to_a_wireguard_client/)
- [DigitalOcean, assign public ipv6 to wireguard clients](https://gist.github.com/MartinBrugnara/cb0cd5b53a55861d92ecba77c80ba729)
- [Wireguard and ipv6](https://www.adyxax.org/blog/2023/02/28/wireguard-and-ipv6/)



