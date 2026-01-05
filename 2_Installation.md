KUBERNETES (CONTAINERD + DOCKER) SETUP — COMMANDS WITH PURPOSE
===============================================================

This file explains what each command does and why it’s needed.
It helps prepare an Ubuntu system as a Kubernetes node (master or worker)
using containerd + Docker as the container runtime.

---------------------------------------------------------------

1) SYSTEM UPDATE AND UPGRADE
----------------------------
sudo apt update && sudo apt upgrade -y

Purpose:
- Updates package lists and upgrades existing packages to the latest versions.
- Ensures a stable, up-to-date system before installing Kubernetes components.


2) DISABLE SWAP (REQUIRED BY KUBELET)
-------------------------------------
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

Purpose:
- Disables swap memory immediately and permanently.
- Kubernetes requires swap to be off for scheduling stability.


3) LOAD KERNEL MODULES FOR CONTAINERS
-------------------------------------
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

Purpose:
- Loads necessary kernel modules for overlay filesystems and bridged networking.
- Ensures container networking and file systems function properly.


4) CONFIGURE SYSTEM NETWORK SETTINGS FOR KUBERNETES
---------------------------------------------------
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system

Purpose:
- Enables packet forwarding and bridge networking.
- Ensures Kubernetes networking (CNI plugins, pods) works properly.


5) INSTALL DOCKER REPOSITORY REQUIREMENTS
-----------------------------------------
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

Purpose:
- Installs necessary utilities (`ca-certificates`, `curl`) and adds Docker’s official repository.
- Downloads and verifies Docker’s GPG key.
- Prepares system to install Docker and containerd packages securely.


6) INSTALL DOCKER ENGINE AND CONTAINERD
---------------------------------------
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

Purpose:
- Installs:
  * docker-ce → Docker Engine (daemon)
  * docker-ce-cli → Docker command-line client
  * containerd.io → container runtime
  * docker-buildx-plugin → advanced build tool
  * docker-compose-plugin → for multi-container apps
- These components together provide Docker and containerd environments.


7) CONFIGURE CONTAINERD FOR SYSTEMD CGROUPS
-------------------------------------------
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

Purpose:
- Generates default containerd configuration file.
- Enables SystemdCgroup to match Kubernetes’ preferred cgroup driver.
- Restarts and enables containerd service to apply changes and start on boot.


8) ADD KUBERNETES REPOSITORY
-----------------------------
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update

Purpose:
- Installs dependencies for HTTPS repositories.
- Adds the Kubernetes stable repository and GPG key.
- Makes kubeadm, kubelet, and kubectl packages available for installation.


9) INSTALL KUBEADM, KUBELET, AND KUBECTL
----------------------------------------
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet

Purpose:
- Installs core Kubernetes tools:
  * kubelet → node agent that runs pods
  * kubeadm → tool for cluster bootstrap/init
  * kubectl → command-line interface
- Holds versions to prevent unintended upgrades.
- Enables and starts kubelet service.


CHECKLIST AFTER INSTALLATION
----------------------------
- Reboot the system.
- Confirm swap is off: `sudo swapon --show`
- Check containerd status: `sudo systemctl status containerd`
- Check kubelet status: `sudo systemctl status kubelet`
- (Optional) Add your user to docker group: `sudo usermod -aG docker $USER`


SUMMARY
--------
This script prepares an Ubuntu system for running Kubernetes by:
✓ Disabling swap  
✓ Setting required kernel modules and sysctl values  
✓ Installing Docker and containerd  
✓ Adding Kubernetes repository  
✓ Installing kubeadm, kubelet, kubectl  
✓ Ensuring runtime and services are configured and running properly

You can now use `kubeadm init` (for control plane) or `kubeadm join` (for worker) to set up your cluster.

---------------------------------------------------------------


note : check if the file crictl.yaml is present in both machine on the location /etc. if not present create crictl.yaml file inside /etc folder in both machine and add rthis content
```
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 2
debug: false
pull-image-on-create: false
``` video no.4 :  12:16 
