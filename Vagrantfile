# -*- mode: ruby -*-
# vi: set ft=ruby :


# Ubuntu server image
IMAGE_NAME = "generic/ubuntu2004" 

# Number of master nodes 
N_M_NODES = 1

# Number of workers nodes (should be at maximum 9)
N_W_NODES = 2

# Container RunTime Engine selection
# Docker=1, containerd=2, CRI-O=3
c_eng = 1

# CNI Network Plugin selection
# Calico=1, Cilium=2, Weave=3, Flannel=4
cni = 1

# Run with noparallel by default
ENV['VAGRANT_NO_PARALLEL'] = 'yes'


Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    config.vm.provider "virtualbox" do |v|
        v.memory = 2048
        v.cpus = 2
        v.gui = false
        # To change the default VirtualBox NAT subnet, uncomment the following line
        # v.customize ["modifyvm", :id, "--natnet1", "10.3/16"]
    end

    config.vm.provider "libvirt" do |v|
        v.nested = true
        v.cpu_mode = "host-model"
        v.memory = 2048
        v.cpus = 2
        v.management_network_name = 'libvirt-custom-network'
        v.management_network_address = '10.10.10.0/24'
    end

    # Setup a nginx load balancer for HA cluster (multi-master nodes)
    if N_M_NODES > 1 then
        config.vm.define "load-balancer" do |lb|
            lb.vm.box = IMAGE_NAME
            lb.vm.network "private_network", ip: "172.16.3.5"
            lb.vm.hostname = "load-balancer"
            lb.vm.provision "ansible" do |ansible|
                ansible.compatibility_mode = "2.0"
                ansible.playbook = "ansible-playbooks/load-balancer.yml"
                ansible.extra_vars = {
                    user: "vagrant",
                    hostname: "load-balancer",
                    node_ip: "172.16.3.5",
                    ansible_python_interpreter:"/usr/bin/python3"
                }
            end
        end
    end

    config.vm.define "master-node-1" do |master|
        master.vm.box = IMAGE_NAME
        master.vm.network "private_network", ip: "172.16.3.10"
        master.vm.hostname = "master-node-1"
        master.vm.provision "ansible" do |ansible|
            ansible.compatibility_mode = "2.0"
            ansible.playbook = "ansible-playbooks/master-playbook.yml"
            ansible.extra_vars = {
                user: "vagrant",
                n_m_nodes: N_M_NODES,
                n_w_nodes: N_W_NODES,
                hostname: "master-node-1",
                node_ip: "172.16.3.10",
                c_eng: c_eng,
                cni: cni,
                ansible_python_interpreter:"/usr/bin/python3"
            }
        end
    end

    # For multi-master nodes clusters, we provide a different ansible playbook to the
    # master node replicas.
    (2..N_M_NODES).each do |i|
        config.vm.define "master-node-#{i}" do |master|
            master.vm.box = IMAGE_NAME
            master.vm.network "private_network", ip: "172.16.3.#{i + 9}"
            master.vm.hostname = "master-node-#{i}"
            master.vm.provision "ansible" do |ansible|
                ansible.compatibility_mode = "2.0"
                ansible.playbook = "ansible-playbooks/master-replica-playbook.yml"
                ansible.extra_vars = {
                    user: "vagrant",
                    n_m_nodes: N_M_NODES,
                    n_w_nodes: N_W_NODES,
                    hostname: "master-node-#{i}",
                    node_ip: "172.16.3.#{i + 9}",
                    c_eng: c_eng,
                    cni: cni,
                    ansible_python_interpreter:"/usr/bin/python3"
                }
            end
        end
    end

    (1..N_W_NODES).each do |i|
        config.vm.define "worker-node-#{i}" do |node|
            node.vm.box = IMAGE_NAME
            node.vm.network "private_network", ip: "172.16.3.#{i + 99}"
            node.vm.hostname = "worker-node-#{i}"
            node.vm.provision "ansible" do |ansible|
                ansible.compatibility_mode = "2.0"
                ansible.playbook = "ansible-playbooks/worker-node.yml"
                ansible.extra_vars = {
                    user: "vagrant",
                    hostname: "worker-node-#{i}",
                    node_ip: "172.16.3.#{i + 99}",
                    c_eng: c_eng,
                    cni: cni,
                    ansible_python_interpreter:"/usr/bin/python3"
                }
            end
        end
    end

end