# Kubernetes playground platform

The objective of this Kubernetes playground platform is to provide a production-like Kubernetes cluster that can be installed either on personal computers and servers. In particular, it allows creating clusters with a single master node, multi-master nodes, and an external etcd cluster, besides choosing the container runtime and CNI to be used within the cluster.

The cluster is spun up using Vagrant, Ansible, and kubeadm.
 - Vagrant is used to create the virtual (using Virtualbox or KVM) masters and workers nodes.
 - Ansible is used to automate a basic software configuration for each node (e.g. installing kubeadm).
 - kubeamd is finally used to bootstrap the Kubernetes nodes (i.e. installing the kube-api-server, kubelet, etc.)


## Hardware Requirements

 - Master and Worker nodes: at least 2 GB of RAM and 2 CPUs (per machine). 
   Check the full requirements here: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin


## Software Requirements

This testbed can be executed either on a Ubuntu or macOS machine. Windows is not currently supported (check out Minikube as an alternative). The following requirements are needed to run the testbed:

 - kubectl [For installation: https://kubernetes.io/docs/tasks/tools/]
 - Vagrant [For installation: https://www.vagrantup.com/downloads]
 - [Optional] Install a plugin to allow copying files from the Host OS to the Guest OS and viceversa: 
       $ vagrant plugin install vagrant-vbguest
       $ vagrant plugin install vagrant-scp
       $ vagrant reload
 - Ansible [For installation: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html]
 - Virtualbox [For installation: https://www.virtualbox.org/wiki/Downloads]
 - KVM [For installation: execute the ubuntu_dependencies file in this repository]


## Installation Issues

#### VirtualBox’s “Kernel Driver Not Installed (rc=-1908)” Error on a Mac

Solution: https://www.howtogeek.com/658047/how-to-fix-virtualboxs-%E2%80%9Ckernel-driver-not-installed-rc-1908-error/

Perhaps a notebook restart is also needed.


## Installation Overview

The current settings allow you to set up a Kubernetes cluster of 1 master node, and N worker nodes (they should be maximum 9). Both the hardware requirements of the master and worker nodes can be customized, specifying the CPU, RAM, and network subnet for each virtual machine. Similarly, the Kubernetes and other components version can be specified in the Ansible provision files.

At the moment, only the Docker container engine is (automatically) supported. In the future, we will add support for containerd and CRI-O container engines.

Instead, four CNI network plugins are currently supported for automatic installation. Just specify the one you'd like to use in the Vagrantfile.


## Kubernetes installation

The following are the steps needed to deploy the Kubernetes cluster.

#### 1. Customize the Vagrantfile

Within the Vagrantfile, in order, you can specify the number of worker nodes, and the CNI network plugin to use (Calico, Cilium, Weave, or Flannel). It is recommended not to change the Ubuntu server image.

#### 2. Spin-up the cluster

From the Kubernetes testbed folder, run the following: 

```bash
vagrant up --provider virtualbox/libvirt 
```

Once the installation is over, Kubernetes will be successfully installed. 


## [Optional] Run kubectl commands from the host

In order to avoid 

#### Option 1.

For this option, the vagrant scp plugin is needed (how-to in the previous "Software Requirements" section). From the testbed directory, run the following: 

```bash
vagrant ssh master-node-1
sudo cat /etc/kubernetes/admin.conf > admin.conf
```

Log out from the guest and run the following:
```bash
scp vagrant@<master-node-ip>:admin.conf .
kubectl --kubeconfig ./admin.conf get nodes
```

After this, you should ssh again in the master-node machine and delete ~/admin.conf.

To create an alias for this command to avoid retyping:
```bash
alias kubectl="kubectl --kubeconfig ./admin.conf"
```

After this, you can run kubectl commands from the host.


#### Option 2. ssh tunnel

Specify the Vagrant guest machine name (e.g. master-node-1) and the command to run (e.g. kubectl get nodes).

```bash
vagrant ssh <guest_machine> -c '<command>'
```

To create an alias for this command to avoid retyping:
```bash
alias kubectl="vagrant ssh <guest_machine> -c"
```

After this, you can run kubectl commands from the host.

To permanently save the alias, add the alias to the `~/.bash_profile` file and run:

```bash
source ~/.bash_profile
```


## Check Kubernetes Installation

Finally, at the end of the installation, we can check that our cluster is up and running as expected.

```bash
kubectl get nodes
kubectl cluster-info
```

For more commands, to retrieve information about pods, services, and other K8s objects check: https://kubernetes.io/docs/reference/kubectl/cheatsheet/


## Nginx Application example

#### Kubernetes Service

From the host machine, run the following:

```bash
kubectl create namespace nginx
kubectl create deployment nginx --image=nginx --namespace=nginx
kubectl expose deployment nginx --type NodePort --port 80 --target-port 80 --namespace=nginx
```

By running the following, you can find out the worker node port assigned to that service (PORTS field, in the range 30000-32767):
```bash
kubectl get svc nginx -n nginx
```

Now you can access the nginx application from the host os at this address: `http://<worker-node-ip>:<node-port>`

#### Port Forwarding

You can also access an application deployed on the cluster through port forwarding from the guest OS to the host OS, though configuring a Kubernetes service is recommended. Hereby the instructions: https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/

An example with the nginx application:
```bash
kubectl create deployment nginx --image=nginx
kubectl get pods -l app=nginx
POD_NAME=$(kubectl get pods -l app=nginx -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:80
```

Now you can access the nginx application from the host os at this address: `http://127.0.0.1:8080`

#### Clean-up

To clean up the previous deployment:

```bash
kubectl delete services nginx
kubectl delete deployments nginx
```

Or, to delete every objects (not namespaces):

```bash
kubectl delete all --all
```

For more examples (e.g. retrieving container logs), check out: https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/13-smoke-test.md


## Application Examples

The following are some application examples that can be deployed on the cluster: 
 - https://github.com/stefanprodan/podinfo
 - https://github.com/kubernetes/examples
 - https://pwittrock.github.io/docs/tutorials/stateless-application/guestbook/
 - Vulnerable deployments: Bust-A-Kube, kube-goat

Other applications are available on Github: TrainTicket, Spring PetClinic, Sock Shop, Movierecommendations, eShop, and Pig-gyMetrics.

