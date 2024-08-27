# Configuration variables
IMAGE_NAME = "bento/ubuntu-22.04"   # Image to use
MEM = 2048                          # Amount of RAM
CPU = 2                             # Number of CPUs
MASTER_NAME="master"                # Master node name
WORKER_NBR = 2                      # Number of workers node

# K8S network config
NODE_NETWORK_BASE = "192.168.56"    # Private network for node communication ..56 est imposé par vagrant
POD_SUBNET = "10.244.0.0/16"        # Private network for inter-pod communication
# HOST_NETWORK_BASE = "192.168.100" # Public network for host communication (NAT)
# SERVICE_SUBNET = "10.96.0.0/12"   # Private network for service communication


# Flannel network config
FLANNEL_NETWORK = "10.244.0.0/16"
FLANNEL_SUBNET = "10.244.0.1/24"
FLANNEL_MTU = 1450
FLANNEL_IPMASQ =true



Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    # RAM and CPU config
    config.vm.provider "virtualbox" do |v|
        v.memory = MEM
        v.cpus = CPU
    end

    # Master node config
    config.vm.define MASTER_NAME do |master|

        # Hostname and network config
        master.vm.box = IMAGE_NAME
        # master.vm.network "public_network", ip: "#{HOST_NETWORK_BASE}.10", bridge: "wlp111s0"
        master.vm.network "private_network", ip: "#{NODE_NETWORK_BASE}.10"
        master.vm.hostname = MASTER_NAME

        # Ansible role setting
        master.vm.provision "ansible" do |ansible|

            # Ansbile role that will be launched
            ansible.playbook = "roles/main.yml"

            # Groups in Ansible inventory
            ansible.groups = {
                "masters" => ["#{MASTER_NAME}"],
                "workers" => ["worker-[1:#{WORKER_NBR}]"]
            }

            # Overload Ansible variables
            ansible.extra_vars = {
                master_node_name: "#{MASTER_NAME}",
                node_name: "#{MASTER_NAME}",

                master_node_ip: "#{NODE_NETWORK_BASE}.10",
                node_ip: "#{NODE_NETWORK_BASE}.10",
                pod_subnet: "#{POD_SUBNET}",
                # service_subnet: "#{SERVICE_SUBNET}",

                flannel_network: "#{FLANNEL_NETWORK}",
                flannel_subnet: "#{FLANNEL_SUBNET}",
                flannel_mtu: "#{FLANNEL_MTU}",
                flannel_ipmasq: "#{FLANNEL_IPMASQ}"
            }
        end
    end

    # Worker node config
    (1..WORKER_NBR).each do |i|
        config.vm.define "worker-#{i}" do |worker|

            # Hostname and network config
            worker.vm.box = IMAGE_NAME
            # worker.vm.network "public_network", ip: "#{HOST_NETWORK_BASE}.#{i + 10}", bridge: "wlp111s0"
            worker.vm.network "private_network", ip: "#{NODE_NETWORK_BASE}.#{i + 10}"
            worker.vm.hostname = "worker-#{i}"

            # Ansible role setting
            worker.vm.provision "ansible" do |ansible|

                # Ansbile role that will be launched
                ansible.playbook = "roles/main.yml"

                # Groups in Ansible inventory
                ansible.groups = {
                    "masters" => ["#{MASTER_NAME}"],
                    "workers" => ["worker-[1:#{WORKER_NBR}]"]
                }

                # Overload Anqible variables
                ansible.extra_vars = {
                    node_ip:  "#{NODE_NETWORK_BASE}.#{i + 10}",
                    node_name: "worker-#{i}",
                    pod_subnet: "#{POD_SUBNET}"
                }
            end
        end
    end
end
