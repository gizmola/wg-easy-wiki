This is an example on how to use WireGuard Easy with nginx, to access it on a HTTPS domain (e.g. `https://wg-easy.myhomelab.com`).

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
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1

  nginx:
    image: weejewel/nginx-with-certbot
    container_name: nginx
    hostname: nginx
    volumes:
      - ~/.nginx/servers/:/etc/nginx/servers/
      - ./.nginx/letsencrypt/:/etc/letsencrypt/
    ports:
      - "80:80/tcp"
      - "443:443/tcp"
    restart: unless-stopped
```

## `~/.nginx/servers/wg-easy.conf`:

```
server {
    server_name ⚠️wg-easy.myhomelab.com;

    location / {
        proxy_pass http://wg-easy:51821/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header Host $host;
    }
}
```

Save these files, edit the variables marked with `⚠️` and run `docker-compose up -d` in the same directory.

Then run once:

```bash
$ docker exec -it nginx /bin/sh
$ cp /etc/nginx/servers/wg-easy.conf /etc/nginx/conf.d/.
$ certbot --nginx --non-interactive --agree-tos -m webmaster@google.com -d ⚠️wg-easy.myhomelab.com
$ nginx -s reload
$ exit
```

Of course, make sure to point `wg-easy.myhomelab.com` to your server's IP address with a DNS A record or DynamicDNS or any other method. Ensure ports `80`, `443`, `51820` are available (e.g. by forwarding them in your router).

That's it!