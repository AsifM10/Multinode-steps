KUBERNETES VM CLEANUP & REINSTALL NOTES
======================================

This document contains step-by-step commands to clean Kubernetes
from VMs and optionally recreate the cluster again.

--------------------------------------------------
SECTION 1: DELETE KUBERNETES RESOURCES
--------------------------------------------------

Delete all deployments:
kubectl delete deploy --all

Delete all services:
kubectl delete svc --all

Delete all jobs:
kubectl delete jobs --all

Delete specific configmap:
kubectl delete configmap builders-config


--------------------------------------------------
SECTION 2: REMOVE KUBERNETES CLUSTER
--------------------------------------------------

Reset Kubernetes cluster (run on BOTH machines – master & worker):
sudo kubeadm reset

Remove kubeconfig directory:
rm -rf .kube/


--------------------------------------------------
SECTION 3: RESTART KUBERNETES CLUSTER (OPTIONAL)
--------------------------------------------------

IF YOU WANT TO RESTART THE CLUSTER AGAIN
{

Create cluster (run ONLY on MASTER):
sudo kubeadm init


After init, run the following commands on MASTER:
--------------------------------------------------
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


Join worker node (run on WORKER):
--------------------------------------------------
sudo kubeadm join 10.210.13.111:6443 --token 883u0c.2hujzb69u9s4k9ah \
--discovery-token-ca-cert-hash sha256:683dd50ac4f226575936da0ddaa656f110439eda4aad62db4503ce398c43af63

NOTE:
This join command appears after running `kubeadm init`
on the master machine. Copy that command and run it
on the worker machine.

}


--------------------------------------------------
SECTION 4: REMOVE DOCKER
--------------------------------------------------

Remove Docker from BOTH machines:
sudo apt-get remove docker docker-engine docker.io docker-compose docker-registry


--------------------------------------------------
SECTION 5: REMOVE KUBERNETES COMPONENTS
--------------------------------------------------

Remove Kubernetes components from BOTH machines:
sudo apt-get purge kubeadm kubectl kubelet kubernetes-cni kube*

Run autoremove to clean unused packages:
sudo apt-get autoremove

Remove kube config directory (on MASTER):
sudo rm -rf ~/.kube


--------------------------------------------------
SECTION 6: CLEAN REMAINING FILES & REPOS
--------------------------------------------------

Run the following commands on BOTH machines:
--------------------------------------------------
sudo rm -rf /etc/docker
sudo rm -rf /usr/bin/docker
sudo rm -rf /usr/share/man/man1/docker.1.gz
sudo rm -rf /etc/apt/sources.list.d/docker.list
sudo rm -rf /etc/kubernetes
sudo rm -rf /etc/apt/sources.list.d/kubernetes.list
sudo rm -rf /usr/libexec/kubernetes
yes | sudo apt-get update


Confirm removal:
--------------------------------------------------
kubectl --version
kubeadm --version


--------------------------------------------------
SECTION 7: UNINSTALL CONTAINERD
--------------------------------------------------

Check containerd version:
containerd --version

Steps (run on BOTH machines):
--------------------------------------------------
1) systemctl status containerd
2) sudo systemctl stop containerd
3) sudo systemctl disable containerd
4) sudo apt purge containerd -y
5) sudo apt purge containerd.io -y
6) sudo apt autoremove -y

Confirm containerd removal:
--------------------------------------------------
containerd --version

