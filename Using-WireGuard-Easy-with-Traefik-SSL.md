This is an example on how to use WireGuard Easy with Traefik, to access it on a HTTPS domain (e.g. `https://vpn.myhomelab.com`).

## `docker-compose.yml`:

```yaml
version: "3.8"
services:
  wg-easy:
    labels:
      # traefik
      - "traefik.enable=true"
      - "traefik.http.services.WireGuardService.loadbalancer.server.port=51821"
      # http to https
      - "traefik.http.routers.WireGuardRoute.service=WireGuardService"
      # ⚠️ Required:
      # Change this to your host's public address
      - "traefik.http.routers.WireGuardRoute.rule=Host(`vpn.myhomelab.com`)"
      - "traefik.http.routers.WireGuardRoute.entrypoints=web"
      - "traefik.http.routers.WireGuardRoute.middlewares=HttpToHttpsRedirectMiddleware"
      # https
      - "traefik.http.routers.WireGuardRouteSSL.service=WireGuardService"
      # ⚠️ Required:
      # Change this to your host's public address
      - "traefik.http.routers.WireGuardRouteSSL.rule=Host(`vpn.myhomelab.com`)"
      - "traefik.http.routers.WireGuardRouteSSL.entrypoints=websecure"
      - "traefik.http.routers.WireGuardRouteSSL.tls.certresolver=MainCertResolver"
    environment:
      # ⚠️ Required:
      # Change this to your host's public address
      WG_HOST: vpn.homelab.com

      # Optional:
      # - PASSWORD=
      # - WG_PORT=51820
      # - WG_DEFAULT_ADDRESS=10.8.0.x
      # - WG_DEFAULT_DNS=1.1.1.1
      # - WG_MTU=1420
      #-  WG_ALLOWED_IPS=
      # - WG_PRE_UP=echo "Pre Up" > /etc/wireguard/pre-up.txt
      # - WG_POST_UP=echo "Post Up" > /etc/wireguard/post-up.txt
      # - WG_PRE_DOWN=echo "Pre Down" > /etc/wireguard/pre-down.txt
      # - WG_POST_DOWN=echo "Post Down" > /etc/wireguard/post-down.txt

    container_name: wg-easy
    image: weejewel/wg-easy
    networks:
      - traefik_network
    volumes:
      - .:/etc/wireguard
    ports:
      - "51820:51820/udp"
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1

  traefik:
    image: traefik:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - traefik_letsencrypt_data:/letsencrypt
    networks:
      - traefik_network
    ports:
      - "80:80"
      - "443:443"
    command:
      - "--providers.docker"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--providers.docker.network=traefik_network"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.MainCertResolver.acme.tlschallenge=true"
      # ⚠️ Change the email to yours (to receive notifications from letsencrypt)
      - "--certificatesresolvers.MainCertResolver.acme.email=email@myhomelab.com"
      - "--certificatesresolvers.MainCertResolver.acme.storage=/letsencrypt/acme.json"

networks:
  traefik_network:
    external: true

volumes:
  traefik_letsencrypt_data:
```
Save file docker-compose.yml, edit the variables marked with ⚠️ and run docker-compose up -d in the same directory.

Of course, make sure to point wg-easy.myhomelab.com to your server's IP address with a DNS A record or DynamicDNS or any other method. Ensure ports 80, 443, 51820 are available (e.g. by forwarding them in your router).

That's it!