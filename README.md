# Kubernetes playground platform

The objective of this Kubernetes playground platform is to provide an environment that can be replicated and exploited to test its security and eventually find possible attacks and vulnerabilities.

## Requirements

The following installation has been tested on a Ubuntu server. It can be either a server running on bare metal, on a hypervisor (Virtual Box or KVM), or in nested virtualization (i.e. running on a KVM VM running inside another KVM/VirtualBox VM). If you do not have an ubuntu server to test the environment, you can spin one up from the ubuntu-servers folder. Keep in mind that, usually, nested virtualization is fairly slow.

## 1) Ubuntu servers initialization

If you already have a ready-to-use Linux server, you can skip this part. The first step is to create the servers (1..N) on which Kubernetes clusters will be deployed. The installation files are in the ubuntu-server folder. You need to install the following dependencies:
 - Vagrant
    - vagrant plugin install vagrant-scp
 - Ansible
 - VirtualBox/libvirt

If you need to create the servers VMs from scratch, you can run the following command within the ubuntu-server folder, specifying (in the Vagrantfile) how many servers you want to sping up, the resources to allocate for each of them and the network subnet:

```bash
vagrant up --provider virtualbox/libvirt
```

Once the server/s is running, you can install the requirements with the following commands (still from the ubuntu-server folder):
```bash
vagrant scp start server-N:/home/vagrant
vagrant ssh server-N
chmod +x start
./start
```

You need to execute those commands for each ubuntu-server you created. After the installation, each server will be able to host a Kubernetes cluster.

## 2) Kubernetes installation

The following commands will set up a Kubernetes cluster (k8s_multi_cluster --- 2 masters, 6 nodes, or k8s_single_cluster --- 1 master, 3 nodes) on VirtualBox or libvirt using Vagrant and Ansible. Before you start, make sure you have installed the following dependencies:
 - Vagrant
 - Ansible
    - ansible-galaxy collection install community.kubernetes
 - VirtualBox/libvirt

Also, remember to tune the configuration of Vagrantfiles (e.g. CPU, RAM) based on your available hardware.

Copy the Kubernetes cluster folder (k8s_multi_cluster or k8s_single_cluster --- using vagrant scp) on your server and run the following:

```bash
vagrant up --provider virtualbox/libvirt 
```

Once the installation is over, Kubernetes will be successfully installed. You can now start playing with it.

## 3) Kubernetes Cluster management

To manage the cluster's VMs we can use vagrant:

Accessing the VMs:
```bash
vagrant ssh machine_name (k8s-master, etc.)
```

Pausing the VMs:

```bash
vagrant halt
```

Restarting the VMs (e.g. for reloading Ansible files):

```bash
vagrant reload
```

## 4) Install an application

Once the Kubernetes cluster is up and running, you can install an application on it.

Application examples: 
 - https://github.com/kubernetes/examples
 - https://pwittrock.github.io/docs/tutorials/stateless-application/guestbook/

Other applications available on Github: TrainTicket, Spring PetClinic, Sock Shop, Movierecommendations, eShop, and Pig-gyMetrics.

Vulnerable deployments: Bust-A-Kube, kube-goat

