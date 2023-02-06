# docker-swarm 4 node raspberry pi cluster
##### This is for home lab use only.  No security or hardening has been configured.
<details>
  <summary>Install OS</summary>

### Install headless raspberry pi os
I used Raspberry PI OS Lite, and balenaEtchter to burn my microsd cards
https://www.raspberrypi.com/software/operating-systems/
- decent write up
https://florianmuller.com/build-a-raspberry-pi-4-docker-swarm-cluster-with-four-nodes-and-deploy-traefik-with-portainer

#### After install run on all nodes
apt update && apt full-upgrade -y\
reboot

##### After reboot
rpi-eeprom-update -a\
reboot

</details>

<details>
  <summary>Configure Router</summary>

I have a linksys wrt3200acm router with dd-wrt.  Under services, give the raspberry pi's a static IP lease. [Your IP addresses will differ]
![Static Lease](./doc/images/dd-wrt-static-lease.png)
If you want your local network to point to the master node of the swarm.  [Your IP addresses will differ]
![Static Lease](./doc/images/dnsmasq.png)
</details>

<details>
  <summary>Install Ansible</summary>

### Install ansible on the master node.  Need to be root [sudo su]
#### Install
apt update\
apt install software-properties-common\
apt-add-repository --yes --update ppa:ansible/ansible\
apt install ansible

#### Generate and copy ssh keys from master/control node [rp1.local] in my case
mkdir -p ~/.ssh && chmod 700 ~/.ssh\
ssh-keygen -t rsa\
ssh-copy-id -i ~/.ssh/id_rsa.pub your_user@rp2.local\
ssh-copy-id -i ~/.ssh/id_rsa.pub your_user@rp3.local\
ssh-copy-id -i ~/.ssh/id_rsa.pub your_user@rp4.local

##### Build host file
nano /etc/ansible/host\
Example content:
```
[control]
rp1.local ansible_connection=local

[workers]
your_user@rp2.local
your_user@rp3.local
your_user@rp4.local

[cube:children]
control
workers
```

##### Test ansbible
ansible cube -m ping
</details>

<details>
  <summary>Install Docker Swarm</summary>

#### Run the playbook
ansible-playbook docker-swarm-install.yml

</details>

<details>
  <summary>Install & Configure NFS Shares</summary>

#### In the playbook file ./nas-mnt-dir.yaml
- Adjust the share paths.  I am using another single raspberry pi with Open Media Vault (OMV), and an external USB3 enclosure with 2 1TB disks.  Within OMV I am using an RSYNC task to keep the main disk backed up, instead of using the software raid, which in some circles is a better way to go.  Also, you will see the path with k3s, as I started with Kubernetes, but switched back to swarm.  Swarm seems simplier to me for what I am doing, but I would still like to learn more about Kubernetes.
#### Run the playbook
ansible-playbook ./nas-mnt-dir.yaml

</details>

<details>
  <summary>Install Traefik & Portainer</summary>

#### In the playbook file ./portainer-traefik/traefik_portainer_swarm.yml
- Adjust the NFS share paths in the volume definitions within the stack.  Also, look at the host name for the portainer traefik definition, and adjust that to the local domain you setteled with...
#### Run the playbook
ansible-playbook ./portainer-traefik/traefik_portainer_swarm.yml

</details>

<details>
  <summary>Install Elasticsearch</summary>

#### In the playbook file ./portainer-traefik/traefik_portainer_swarm.yml
- Adjust the NFS share paths in the volume definitions within the stack.  Also, look at the host name for the portainer traefik definition, and adjust that to the local domain you setteled with...
#### Run the playbook
ansible-playbook ./portainer-traefik/traefik_portainer_swarm.yml

</details>