If you don't want to use docker, then install wireguard, nodejs and npm from your package manager and then, run the following.
<pre>
echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf
echo net.ipv4.conf.all.src_valid_mark=1 >> /etc/sysctl.conf
sysctl -p
git clone https://github.com/WeeJeWel/wg-easy
cd wg-easy
mv src /app
cd /app
npm ci --omit=dev
cp node_modules ..
ufw allow 51821/tcp # (webui) Only for users of the UFW firewall
ufw allow 51820/udp # (wireguard listening port) Only for users of the UFW firewall
cd -
================================================================================
wget -O /etc/systemd/system/wg-easy.service https://envs.sh/QQK.txt  <--broke??
================================================================================
nano /etc/systemd/system/wg-easy.service # Replace everything that is marked as 'REPLACEME' and tweak it to your liking
systemctl daemon-reload
systemctl enable --now wg-easy.service
systemctl start wg-easy.service
</pre>

To upgrade do the following
<pre>
git clone https://github.com/WeeJeWel/wg-easy # do this if you dont have the repository cloned aldready
cd wg-easy
git pull
rm -rf /app /node_modules
mv src /app
cd /app
npm ci --omit=dev
cp node_modules ..
</pre>