
#IMAGE_NAME = "bento/ubuntu-16.04"
#MEMORY_MB = 1024

IMAGE_NAME = "ubuntu/bionic64" # 18.04 LTS
MEMORY_MB = 8192
#WORKERS = 3
WORKERS = 1
#MASTERS = 3 - not yet implemented (controller join)
MASTERS = 1
#ETCD = 0 - not yet implemented (separate nodes)
#
#

# Be careful that this matches with values in playbooks:
# master: 192.168.50.11
HOST_SUBNET="192.168.50"

def create_node(config, node_name, node_ip, playbook)
    config.vm.define node_name do |node|
        node.vm.box = IMAGE_NAME
        node.vm.network "private_network", ip: node_ip
        node.vm.hostname = node_name
        node.vm.provision "shell",
            inline: "apt-get update; apt-get install -y ansible"
        node.vm.provision "ansible" do |ansible|
            ansible.playbook = playbook
            ansible.extra_vars = {
                node_ip: node_ip,
            }
        end
    end
end

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    config.vm.provider "virtualbox" do |v|
        v.memory = MEMORY_MB
        v.cpus = 2
    end
      
    (1..MASTERS).each do |i|
        master_playbook="kubernetes-setup/master-playbook.yml"
        nodeip="#{HOST_SUBNET}.1#{i}"
        if MASTERS > 1
            nodename="master#{i}"
        else
            nodename="master"
        end
        create_node(config, nodename, nodeip, master_playbook)
    end

    (1..WORKERS).each do |i|
        worker_playbook="kubernetes-setup/worker-playbook.yml"
        nodeip="#{HOST_SUBNET}.#{i + 20}"
        nodename="worker#{i}"
        create_node(config, nodename, nodeip, worker_playbook)
    end
end
