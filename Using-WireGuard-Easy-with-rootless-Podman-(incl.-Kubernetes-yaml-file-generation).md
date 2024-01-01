#### Raspberry Pi 4B 4GB, DietPi v8.0.2 (thin version of Raspbian OS)

I wanted to use this project via podman instead of Docker. When using [podman-compose](https://github.com/containers/podman-compose) (in [this issue](https://github.com/WeeJeWel/wg-easy/issues/27)) I couldn't get it to work properly. My steps for how I implemented this into Podman as well as how I created a Kubernetes yaml file (for using with podman kube play, like docker-compose) are mostly taken from [this article](https://www.redhat.com/sysadmin/compose-podman-pods) (which is a good place to start for converting docker-compose files to kubernetes/podman files).

The kube file I generated can be found in the PR (#181). The kube file needs the same attention regarding environment variables as the docker-compose file if someone wishes to build the pod from the kube file in podman - just uncomment/fill in the variables as you would in the docker-compose file.

There's 3 parts to this:
1. Recreating the wg-easy project as a rootless pod in podman
2. Generating the kube file for the pod so that podman can create the pod from a file, similar to a docker-compose file
3. Creating and linking systemd services to the pod (requires sudo), so that the pods start automatically on boot as their own service


### 1. Recreating the wg-easy project as a rootless pod in podman

Clone the wg-easy repo, cd into the folder.

It seems that I needed to install "slirp4netns" in order to get this to work for me. [slirp4netns](https://github.com/rootless-containers/slirp4netns) is for user-mode networking for unprivileged network namespaces. After installing slirp4netns, I recreated the wg-easy project into a rootless podman pod using the following commands below:

```
podman pod create --name=wg-easy_pod \
                 -p=<WG_PORT>:51820/udp \
                 -p=<WEB_UI_PORT>:51821/tcp
```

`WG_PORT` refers to the WG_PORT variable - the public UDP port of your VPN server. WireGuard will always listen on 51820 inside the Podman container.
`WEB_UI_PORT` refers to the port in which you wish to access the web UI through.

```
podman run -d --restart=unless-stopped \
                 --pod=wg-easy_pod \
                 --name=wg-easy \
                 --volume=.:/etc/wireguard:Z \
                 --cap-add=NET_ADMIN,NET_RAW,SYS_MODULE \
                 --cap-drop=MKNOD,AUDIT_WRITE \
                 --sysctl=net.ipv4.ip_forward=1 \
                 --sysctl=net.ipv4.conf.all.src_valid_mark=1 \
                 -e WG_HOST= \
                 -e PASSWORD= \
                 -e WG_PORT= \
                 -e ...
                 ghcr.io/wg-easy/wg-easy:latest
```

Run podman pod ps to check if the pod was created successfully and has a good state. Visit the WG_PORT of your Wireguard host now and confirm that it is working correctly.


### 2. Generating the kube file for the pod so that podman can create the pod from a file, similar to a docker-compose file

`podman generate kube > wg-easy_kubefile.yaml`

This kube file will contain a lot of environment variables that are unnecessary to define and may even cause issues in the future. In the env section it is safe to remove all env entires except for the container entry and of course the ones you wish to use for this wg-easy project (such as WG_HOST, WG_PASSWORD, etc.).

Capabilities may also need to be added as they seem to change for me. The capabilities add section should look like:

```
      capabilities:
        add:
        - CAP_NET_ADMIN
        - CAP_NET_RAW
        - CAP_SYS_MODULE
```

When you've thinned the kube file environment variables out as you wish and made sure the capabilities look correct, stop and remove the pod & container from podman, as well as the images, so we can test if podman can successfully build the wg-easy pod from the kube file you've generated.

```
podman pod stop wg-easy_pod
podman pod rm wg-easy_pod
podman image ls
podman image rm <wg-easy image>
```

Generate the pod from the kube file with `podman play kube wg-easy_kubefile.yaml`

Run podman pod ps to check if the pod was created successfully and has a good state. Visit the WG_PORT of your Wireguard host now and confirm that it is working correctly.


### 3. Creating and linking systemd services to the pod (requires sudo), so that the pods start automatically on boot as their own service

**To successfully generate systemd services for the wg-easy_pod pod (two service files, one for the pod and one for the container within the pod), you will need to have created the pod and container via the podman pod create/podman run commands (following step 1). Systemd service files cannot be generated from pods that were created from kube files unfortunately.**

With your wg-easy pod running and working fine, use `podman generate systemd -f -n wg-easy_pod --new` to generate systemd service files. Place these files in /etc/systemd/system and confirm them via `systemctl list-unit-files | grep wg-easy`.

Then enable them and reboot (make sure you're using the correct service file names!):

```
sudo systemctl enable pod-wg-easy_pod.service
sudo systemctl enable container-wg-easy.service
sudo shutdown -r now
```

After a reboot your pods should be working fine. I haven't tested this extensively, everything I know is written down here.