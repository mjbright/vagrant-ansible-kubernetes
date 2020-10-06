
IMAGE_NAME = ENV['IMAGE_NAME']
MEMORY_MB  = ENV['MEMORY_MB'].to_i
CPUS       = ENV['CPUS'].to_i

NUM_MASTERS = ENV['NUM_MASTERS'].to_i
NUM_WORKERS = ENV['NUM_WORKERS'].to_i

HOST_SUBNET_PREFIX = ENV['HOST_SUBNET_PREFIX']

VM_NAME_PREFIX     = ENV['VM_NAME_PREFIX']

puts "NUM_MASTERS=#{NUM_MASTERS} NUM_WORKERS=#{NUM_WORKERS}"

def worker_name_ip(i)
    node_ip="#{HOST_SUBNET_PREFIX}.#{i + 20}"
    node_name="worker#{i}"
    return node_name, node_ip
end

def master_name_ip(i)
    node_ip="#{HOST_SUBNET_PREFIX}.1#{i}"
    node_name="master"
    if NUM_MASTERS > 1
        node_name="master#{i}"
    end
    return node_name, node_ip
end

# This applies to all nodes - because of use of  config.vm.network !!
# puts shows this on all nodes: Forwarding ports for master 6443 -> 6443 (auto_correct)
def expose_port(config, node_name, node_ip, vm_ext_port, vm_int_port)
  # (prints on ssh connection!) puts "Forwarding ports for #{node_name} #{vm_int_port} -> #{vm_ext_port} (auto_correct)"
  config.vm.network "forwarded_port", guest: vm_int_port, host: vm_ext_port, auto_correct: true
end

def create_node(config, node_name, node_ip, playbook)
    config.vm.define node_name do |node|
        node.vm.box = IMAGE_NAME

        puts "create_node(config, node_name=#{node_name}, node_ip=#{node_ip}, playbook=#{playbook}"

        # Force specific name for VM to avoid VERR_CFGM_NOT_ENOUGH_SPACE errors
        # (because default name built up from directory/date/time is too long!)
        #node.vm.name = node_name
        node.vm.provider :virtualbox do |v|
            #v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
            #v.customize ["modifyvm", :id, "--memory", 512]
            #vm_name_prefix
            vbox_node_name=node_name
            if VM_NAME_PREFIX != ""
                vbox_node_name="#{VM_NAME_PREFIX}_#{node_name}"
                v.customize ["modifyvm", :id, "--name", vbox_node_name]
            end
        end

        #if node_name == "master"
            #config.vm.network "forwarded_port", guest: 6443, host: 6443
        #end

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

def run_playbook_all_nodes(config, playbook)
    #(1..NUM_MASTERS).each do |i|
    (1..1).each do |i|
        node_name, node_ip=master_name_ip(i)
        run_playbook(config, node_name, node_ip, playbook)
    end
    (1..NUM_WORKERS).each do |i|
        node_name, node_ip=worker_name_ip(i)
        run_playbook(config, node_name, node_ip, playbook)
    end
end

def run_playbook(config, node_name, node_ip, playbook)
    config.vm.define node_name do |node|
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
        v.cpus   = CPUS
    end
      
    (1..NUM_MASTERS).each do |i|
        master_playbook="kubernetes-setup/master-playbook.yml"
        node_name, node_ip=master_name_ip(i)
        create_node(config, node_name, node_ip, master_playbook)
    end

    (1..NUM_WORKERS).each do |i|
        worker_playbook="kubernetes-setup/worker-playbook.yml"
        node_name, node_ip=worker_name_ip(i)
        create_node(config, node_name, node_ip, worker_playbook)
    end

    # # ON single MASTER:
    # (1..1).each do |i|
        # playbook="kubernetes-setup//etc-hosts-playbook.yml"
        # node_name, node_ip=master_name_ip(i)
        # run_playbook(config, node_name, node_ip, playbook)
    # end

    # ON single MASTER:
    (1..1).each do |i|
        playbook="kubernetes-setup/all-ssh-keys-playbook.yml"
        node_name, node_ip=master_name_ip(i)
        run_playbook(config, node_name, node_ip, playbook)

        playbook="kubernetes-setup/tools-playbook.yml"
        node_name, node_ip=master_name_ip(i)
        run_playbook(config, node_name, node_ip, playbook)

        expose_port(config, node_name, node_ip, 6443, 6443)
    end
    (1..NUM_WORKERS).each do |i|
        playbook="kubernetes-setup/all-ssh-keys-playbook.yml"
        node_name, node_ip=worker_name_ip(i)
        run_playbook(config, node_name, node_ip, playbook)
    end
end

