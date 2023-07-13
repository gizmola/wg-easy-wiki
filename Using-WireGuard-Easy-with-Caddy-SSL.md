This is an example on how to use WireGuard Easy with [Caddy](https://caddyserver.com), to access it on an HTTPS domain (e.g. `https://wg-easy.myhomelab.com`).

## `docker-compose.yml`:

```yaml
version: "3.8"

services:
  wg-easy:
    environment:
      # ⚠️ Change the server's hostname (clients will connect to):
      - WG_HOST=wg-easy.myhomelab.com

      # ⚠️ Change the Web UI Password:
      - PASSWORD=foobar123
    image: weejewel/wg-easy
    container_name: wg-easy
    hostname: wg-easy
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

  caddy:
    image: caddy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    volumes:
      - $PWD/Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config

volumes:
  caddy_data:
  caddy_config:
```

## `Caddyfile` (Same location as docker-compose.yml):

```
⚠️wg-easy.myhomelab.com {
  reverse_proxy wg-easy:51821
}
```

Save these files, edit the variables marked with `⚠️` and run `docker-compose up -d` in the same directory.
Caddy takes care of certificate generation and renewal automatically.

Of course, make sure to point `wg-easy.myhomelab.com` to your server's IP address with a DNS A record or DynamicDNS or any other method. Ensure ports `80`, `443`, `51820` are available (e.g. by forwarding them in your router).

That's it!