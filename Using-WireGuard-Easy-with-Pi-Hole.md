This is an example on how to use WireGuard Easy with Pi-hole.

By default, all connected clients will use Pi-Hole as DNS server.

## `docker-compose.yml`:

```yaml
version: "3.8"

services:
  wg-easy:
    environment:
      # ⚠️ Change the server's hostname (clients will connect to):
      - WG_HOST=myhost.com

      # ⚠️ Change the Web UI Password:
      - PASSWORD=foobar123

      # 💡 This is the Pi-Hole Container's IP Address
      - WG_DEFAULT_DNS=10.8.1.3
      - WG_DEFAULT_ADDRESS=10.8.0.x
    image: weejewel/wg-easy
    container_name: wg-easy
    volumes:
      - ~/.wg-easy:/etc/wireguard
    ports:
      - "51820:51820/udp"
      - "51821:51821/tcp"
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1
    networks:
      wg-easy:
        ipv4_address: 10.8.1.2

  pihole:
    image: pihole/pihole
    container_name: pihole
    environment:
      # ⚠️ Change the Web UI Password:
      - WEBPASSWORD=foobar123
    volumes:
      - '~/.pihole/etc-pihole:/etc/pihole'
      - './.pihole/etc-dnsmasq.d:/etc/dnsmasq.d'
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "5353:80/tcp"
    restart: unless-stopped
    networks:
      wg-easy:
        ipv4_address: 10.8.1.3

networks:
  wg-easy:
    ipam:
      config:
        - subnet: 10.8.1.0/24
```

Save this file, edit the variables marked with `⚠️` and run `docker-compose up -d` in the same directory. That's it!