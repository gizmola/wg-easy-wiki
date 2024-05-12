# Description

This playbook is designed to automate the installation of Docker and Wireguard Easy on your virtual machine using Ansible.

***

# Installing Ansible

To begin, install Ansible by following the official [documentation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

***

# Downloading the Playbook

Clone the repository containing the Wireguard playbook:

Copy code
```bash
git clone https://gitlab.com/CTPEJIKuH/wireguard-playbook
```
```bash
cd wireguard-playbook
```
# Configuring Variables and VM IP Address

In the inventories/prod/inventory.yml file, specify the IP address of your virtual machine:
```yaml
all:
  children:
    prod:

wireguard_addresses:
  hosts:
    your-vm:
      ansible_host: 123.45.67.89
```
Instead of your-vm, specify the name of your virtual machine.

Instead of 123.45.67.89, you need to specify the IP of your virtual machine.

In the inventories/prod/group_vars/all.yml file, set the password for the Wireguard web interface:
```bash
wg_easy_password: "mypassword"
```
Replace mypassword with your actual password.

Other variables are also available in the role that can be configured. 
Details can be found [here](https://gitlab.com/CTPEJIKuH/wireguard-playbook/-/blob/main/roles/wireguard/README.md?ref_type=heads).
The playbook also provides backup functionality.

***

# Entry Point
At the end of the Ansible run, a message like the following will be displayed your UI address:

```
TASK [wireguard : Output UI address] *******************************************************************************
ok: [your-vm] => {
    "msg": "UI address: http://123.45.67.89:51821"
}
```

***

# Example Playbook
Deployment with a check:
```bash
ansible-playbook -i inventories/prod main.yml -D -C
```
Deployment without check:
```bash
ansible-playbook -i inventories/prod main.yml -D
```