`cp -r node_modules ..`
works better

don't forget to change in MASQUERADE rule interface and ip in /app/config.js.
In my case i used ens3 instead eth0:

`iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o ens3 -j MASQUERADE`

while upgrading don't pull from git without deleting wg-easy dir