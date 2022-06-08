Simply set the environment variable WG_DEFAULT_DNS to the IP address of your Pi-hole server, e.g. 192.168.0.2.

Example:
<pre>
version: "3"
networks:
  private_network:
    ipam:
      driver: default
      config:
        - subnet: 10.2.0.0/24

services:
  wireguard:
    depends_on: pihole
    image: weejewel/wg-easy:latest
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - TZ=Asia/Barnaul # Change to your timezone
      - PASSWORD=password
      - WG_HOST=0.0.0.0 # Change to your server ip
      - WG_DEFAULT_DNS=10.2.0.100
    volumes:
      - ./wireguard:/etc/wireguard
    ports:
      - "51820:51820/udp"
      - "51821:51821/tcp"
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped
    networks:
      private_network:
        ipv4_address: 10.2.0.50

  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    restart: unless-stopped
    hostname: pihole
    environment:
      TZ: "Asia/Barnaul" # Change to your timezone
      WEBPASSWORD: "password"
      FTLCONF_REPLY_ADDR4: 10.2.0.100
    volumes:
      - ./pihole/etc-pihole/:/etc/pihole/
      - ./pihole/etc-dnsmasq.d/:/etc/dnsmasq.d/
    networks:
      private_network:
        ipv4_address: 10.2.0.100
</pre>