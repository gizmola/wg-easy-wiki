In order for WireGuard Easy to run in a container, it needs NET_RAW to be enabled. Docker enables it by default but Podman does not.

# Run with Podman
Add `--cap-add=NET_RAW` to the `podman run` command like so:
```sh
podman run -d \
  --name=wg-easy \
  -e WG_HOST=🚨YOUR_SERVER_IP \
  -e PASSWORD=🚨YOUR_ADMIN_PASSWORD \
  -v ~/.wg-easy:/etc/wireguard \
  -p 51820:51820/udp \
  -p 51821:51821/tcp \
  --cap-add=NET_ADMIN \
  --cap-add=SYS_MODULE \
  --cap-add=NET_RAW \
  --sysctl="net.ipv4.conf.all.src_valid_mark=1" \
  --sysctl="net.ipv4.ip_forward=1" \
  --restart unless-stopped \
  ghcr.io/wg-easy/wg-easy
```

# Run with podman-compose
Add `NET_RAW` to `the cap_add` list.

```yaml
version: "3.8"
services:
  wg-easy:
    environment:
      # ⚠️ Required:
      # Change this to your host's public address
      - WG_HOST=raspberrypi.local

      # Optional:
      # - PASSWORD=foobar123
      # - WG_PORT=51820
      # - WG_DEFAULT_ADDRESS=10.8.0.x
      # - WG_DEFAULT_DNS=1.1.1.1
      # - WG_MTU=1420
      # - WG_ALLOWED_IPS=192.168.15.0/24, 10.0.1.0/24
      # - WG_PRE_UP=echo "Pre Up" > /etc/wireguard/pre-up.txt
      # - WG_POST_UP=echo "Post Up" > /etc/wireguard/post-up.txt
      # - WG_PRE_DOWN=echo "Pre Down" > /etc/wireguard/pre-down.txt
      # - WG_POST_DOWN=echo "Post Down" > /etc/wireguard/post-down.txt
      
    image: ghcr.io/wg-easy/wg-easy
    container_name: wg-easy
    volumes:
      - .:/etc/wireguard
    ports:
      - "51820:51820/udp"
      - "51821:51821/tcp"
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
      - NET_RAW
      - SYS_MODULE
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1

```

# Troubleshooting

If the container isn't working as expected, try attaching it to watch its output (see `--attach` [here](https://docs.podman.io/en/latest/markdown/podman-start.1.html))

## Loading kernel modules

Try loading these kernel modules on the host machine, if they haven't already. e.g. `sudo modprobe iptable_filter`
```
ip_tables
iptable_filter
iptable_nat
wireguard
xt_MASQUERADE
```
See [this issue](https://github.com/containers/podman/issues/15120#issuecomment-1397571841) for more info.

## Set podman network MTU

Especially in rootless containers, if WireGuard Easy claims to be connected to a client but nothing loads over the network, you may need to adjust the MTU (maximum transmission unit) for your podman network. For example, running `podman network create --opt mtu=1500` would create a network with an MTU of 1500. Then, recreate your podman container to use that network. See [here](https://github.com/containers/podman/issues/15120#issuecomment-1369386865) for additional context on this issue and [here](https://docs.podman.io/en/latest/markdown/podman-network.1.html) for documentation on managing networks with podman.