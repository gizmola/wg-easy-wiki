# Problems after starting the Container?
## The general solution may be that your Linux Distribution does not have Wireguard support installed.


If you've configured everything correctly, started up the Easy Wireguard Container and see errors like this, you may be able to remedy the issue by installing required Wireguard Packages.

Logs like this one reflect this problem:

## Fixing the issue on RHEL/Centos/Fedora/Alma or Rocky Linux

# Follow instructions for your Distro to install the epel repository for your Distro
This example comes from an Install to an Alma Linux 8 Server

```
wg-easy  | $ wg-quick down wg0
wg-easy  | $ wg-quick up wg0
wg-easy  | Error: Command failed: wg-quick up wg0
wg-easy  | [#]
wg-easy  | [#] ip link add wg0 type wireguard
wg-easy  | [#] wg setconf wg0 /dev/fd/63
wg-easy  | [#] ip -4 address add 10.8.0.1/24 dev wg0
wg-easy  | [#] ip link set mtu 1420 up dev wg0
wg-easy  | [#] iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE; iptables -A INPUT -p udp -m udp --dport 51820 -j ACCEPT; iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT;
wg-easy  | modprobe: can't change directory to '/lib/modules': No such file or directory
wg-easy  | modprobe: can't change directory to '/lib/modules': No such file or directory
wg-easy  | iptables v1.8.10 (legacy): can't initialize iptables table `nat': Table does not exist (do you need to insmod?)
```

## After installing EPEL, try to install kmod-wireguard and wireguard-tools packages
```
 sudo yum install kmod-wireguard wireguard-tools
```
Reboot after installation!  (sudo shutdown -h now)

## Still getting modprobe: can't change directory error?

```
sudo modprobe ip_tables
```