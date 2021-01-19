# raspberry-pi-k8-cluster

## Prep
```bash
# buy rasp pi 4 4GB, powersupply, cluster case optional

# Download Raspberry Pi Imager from https://www.raspberrypi.org/software/
# install ubuntu server LTS 20.04 arm64
```
## Install Microk8
```bash
# connect to raspi using default password 'ubuntu'
ssh ubuntu@192.168.1.116

# connect again as connection closes with the new password
ssh ubuntu@192.168.1.114

# update ubuntu repos
sudo apt update

# update all packages
sudo apt upgrade

# change hostname (restart not required) Replace <node-name> with master, node01, node 02 on the other rasp
sudo hostnamectl set-hostname rasp-k8-master
sudo hostnamectl set-hostname "Raspberry PI k8 Master" --pretty

# verify changes
hostnamectl

# configure a fix ip for the node
# hint: use 'set paste' in vim to retain the white spaces
sudo vi /etc/netplan/01-netcfg.yaml

### file content ->
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.5/24
      gateway4: 192.168.1.1
      nameservers:
          addresses: [8.8.8.8, 1.1.1.1]
          
          
# apply the fixed ip      
sudo netplan apply

# verify ip
ip addr show dev eth0

```

## Install and Configure Microk8
```bash

# enable c-groups https://microk8s.io/docs/install-alternatives#heading--arm
sudo vi /boot/firmware/cmdline.txt

# add this (ONE COMMAND??)
cgroup_enable=memory cgroup_memory=1

# reboot
sudo reboot

# install microk8s https://microk8s.io/docs
sudo snap install microk8s --classic --channel=1.20

# join groups
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube

# reenter session, needs the password
su - $USER

# check k8 status
microk8s status --wait-ready

# create alias in bashrc and source the bashrc file to make it available in the current session
echo 'alias kubectl="microk8s kubectl"' >> ~/.bashrc
source ~/.bashrc

# repeat above steps to setup node 01 and node 02

# adding other nodes to k8 https://microk8s.io/docs/clustering
# connect to master node using ssh and run
microk8s add-node

# use the output from the above command and run on the node
microk8s join <IP>:25000/<TOKEN>

# check nodes
kubectl get nodes

```


## Enable Kubernetes Dashboard
```bash
# connect to master node via ssh
# enable dashboard https://virtualizationreview.com/articles/2019/01/30/microk8s-part-2-how-to-monitor-and-manage-kubernetes.aspx
microk8s.enable dns dashboard ingress

# see what has been created
kubectl get all -n kube-system

# do a temporary port forwarding
microk8s kubectl port-forward -n kube-system service/kubernetes-dashboard 10443:443 --address 0.0.0.0

# get config file
kubectl config view

# copy content and create config file on your local machine

# access dashboard on local machine
# https://{MASTER_NODE_IP_address}:10443/
# use above config file to connect


# alternative to retrieve token
token=$(microk8s kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
microk8s kubectl -n kube-system describe secret $token

```


## Expose k8 Cluster
```bash
# configure dyndns on router
# and port forwarding for 443 and 80 TCP

# enable storage
microk8s.enable storage

# enable prometheus
# Default (user/pass: admin/admin)
microk8s.enable prometheus

```
