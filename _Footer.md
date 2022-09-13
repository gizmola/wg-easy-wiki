`cp -r node_modules ..`
works better

don't forget to enable MASQUERADE (set your int and ip):
`iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o ens3 -j MASQUERADE`

while upgrading the src dit don't pull from git. recommended to del wg-easy dir before update