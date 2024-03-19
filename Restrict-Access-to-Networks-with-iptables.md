This is an example of how to use iptables through the WG_POST_UP & WG_POST_DOWN environment variables to restrict LAN access for connected clients.

I'll break this into two examples for clarity.

## Block LAN access for all connected clients while still allowing internet access:
```
- WG_POST_UP=iptables -I FORWARD -i wg0 -d 192.168.X.0/24 -j REJECT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
- WG_POST_DOWN=iptables -I FORWARD -D wg0 -d 192.168.X.0/24 -j REJECT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```
Replace `192.168.X.0/24` with your local LAN subnet.

You can add multiple subnets separated by a comma (e.g. `192.168.X.0/24,172.X.0.0/16`) iptables will create multiple rules for each destination.

## Block LAN access except for specific clients while still allowing internet access:
```
- WG_POST_UP=iptables -I FORWARD -i wg0 -d 192.168.X.0/24,172.X.0.0/16 -j REJECT; iptables -I FORWARD -i wg0 -s 10.8.0.X -d 192.168.X.0/24 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
- WG_POST_DOWN=iptables -I FORWARD -D wg0 -d 192.168.X.0/24,172.X.0.0/16 -j REJECT; iptables -I FORWARD -D wg0 -s 10.8.0.X -d 192.168.X.0/24 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```
Building on the first example we're blocking access to 192.168.X.0/24 & 172.X.0.0/16 for any clients connecting.

This time there's an entry added in the middle allowing a specific client access to the otherwise blocked subnet. Change `10.8.0.X` to your clients IP from wg_easy's dashboard. _Note: In my testing even if a different client changes their WireGuard settings to this IP they will no longer have access as there's a mismatch of IP to client list server side._

If you want to allow access to specific IPs (like servers) remove the /24 (e.g. `192.168.X.2`)

You can add multiple subnets/client IPs separated by a comma the same way explained in the previous example.

## Other notes:
The WG_POST_DOWN environment variable is essentially the same line as WG_POST_UP except we're substituting the `-i` for `-D` to delete the entry.

You should be adding these two lines under `environment:` in your docker-compose.yml file.