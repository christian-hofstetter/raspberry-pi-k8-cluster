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
# The authenticity of host '192.168.1.116 (192.168.1.116)' can't be established.
ECDSA key fingerprint is SHA256:0YC27d9uytHLFb5k9MdQtqTRkplCt2gni4VKQzDzrOI.
Are you sure you want to continue connecting (yes/no)? yes

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

# enable c-groups https://microk8s.io/docs/install-alternatives#heading--arm
sudo vi /boot/firmware/cmdline.txt

# add this (ONE COMMAND??)
cgroup_enable=memory cgroup_memory=1

# reboot
sudo reboot

# install microk8s https://microk8s.io/docs
sudo snap install microk8s --classic --channel=1.19

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
```