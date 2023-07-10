# MlOps_Project_Setup
# Table of Contents
- Kubeflow
- Elyra
- Katib
- Kubeflow set up
- Kubeadm set up
- Cluster configuration
- Start Elyra
- Start Katib


# Let's define Kubeflow
- Kubeflow is an open-source platform designed to make it easier to deploy and manage machine learning (ML) workflows on Kubernetes, an open-source container orchestration system.
  
![téléchargement (10)](https://github.com/SkanderBahrini/Kubeflow-setup/assets/74383561/6a1b2b77-a5be-4f94-a559-d651a375d644)

# Kubeflow offers a set of external add-ons:

# Elyra
- Elyra is an open-source project that provides a set of tools and extensions for developing and running machine learning (ML) pipelines in various environments. It focuses on simplifying the creation and deployment of ML pipelines by providing a visual interface and a collection of pre-built components.


![48364812 (2)](https://github.com/SkanderBahrini/Kubeflow-setup/assets/74383561/50018c9f-697d-4b63-b3e5-82f34bd05f67)

# Katib
- Katib is an open-source project that provides automated hyperparameter tuning and optimization for machine learning (ML) models. It is designed to help data scientists and ML practitioners efficiently search for the best combination of hyperparameters that optimize the performance of their models.

![téléchargement (11)](https://github.com/SkanderBahrini/Kubeflow-setup/assets/74383561/51fffa7e-3188-4325-b149-bd9408402364)


# Kubeflow-Setup
This repository will resume my capstone project entitled "Machine learning pipeline automation using Mlops tool Kubeflow".

The project was done using a VMware virtual machine with the following settings:

![Virtual Machine setup](https://github.com/SkanderBahrini/Kubeflow-setup/assets/74383561/897ef97f-6fab-4903-a112-5656d49bc594)

We used Ubuntu 20.04 image in our machine.

The entire infrastructure was built on top of a single-node Kubeadm cluster.

The following instruction will guide you throughout the setup process:

# Set up Docker
Source: https://docs.docker.com/engine/install/ubuntu/
First, we need to install Docker on our virtual machine using the following commands.
- Update the packages list and install the required packages
```
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
```
- Add Docker’s official GPG key:
```
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
- Use the following command to set up the repository:
```
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
- Update Package
```
sudo apt-get update
```
- List the available versions:
```
apt-cache madison docker-ce | awk '{ print $3 }'
```
- Install the required version (replace version)
```
VERSION_STRING=5:24.0.0-1~ubuntu.22.04~jammy
sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
```
- Install Docker's latest version
```
sudo apt install docker.io -y
```
- Once Docker is installed
- Restart Docker
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```
- Enable configuration systemctl
```
sudo modprobe overlay
sudo modprobe br_netfilter
```
# SETUP KUBEADM
- Kubeadm is Kubernetes project that allows users to set up a  boptstraped version of Kubernetes clusterson their serbverz
source: https://phoenixnap.com/kb/install-kubernetes-on-ubuntu

- Install required packages
```
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo apt-get install -y apt-transport-https
```
- Install Kubernetes tools
```
sudo apt install kubeadm=1.22.10-00 kubelet=1.22.10-00 kubectl=1.22.10-00
```
- Download the Google Cloud public signing key:
```
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
```
- Add the Kubernetes apt repository:
```
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
- Set up the Kubeadm cluster (you can add the flags if you want to add specific conditions refer to kubeadm documentation https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/)
```
sudo kubeadm init
```
- set up regular access
```
mkdir -p $HOME/.kube
sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
- check cluster status
```
kubectl cluster-info
```
# Configure the Kubeadm cluster

 :triangular_flag_on_post: We need to untaint our master node allowing creation of pods inside it

- Configure the Container network interface allowing pods communication. In this study, we will use Calico as Plugin using the following commands.
  :exclamation: You need to configure the custom-resources.yaml according to your CIDR configuration.
```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.1/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.1/manifests/custom-resources.yaml
```

- Configure cluster storage using a storage class that will contain all data related to our cluster. In this study we use a local storage
  You cant find below the manifest file:

```
kubectl create -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml

```
- We Mount the volume to our cluster
```
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

```
# Start Using Elyra

- To use Elyra we need to access Kubeflow Dashboard and access to notebook servers.
  
- Once We access it we need to choose between two types of images as explained in the blow activity diagram. 

- We will use the following image from the Dockerhub
```
elyra/kf-notebook
```
- We need to configure the runtime image
https://elyra.readthedocs.io/en/latest/user_guide/runtime-conf.html

- Once the Notebooks server is ready we can obtain a pipeline as follows:![Capture d’écran 2023-05-17 163505](https://github.com/SkanderBahrini/Kubeflow-setup/assets/74383561/9c315f0e-d0cd-4d08-8f20-178707b0d832)

# Use Katib
