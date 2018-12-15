# -*- mode: ruby -*-
# vi: set ft=ruby :

servers = [
    {
        :name => "k8s-head",
        :type => "master",
        :box => "geerlingguy/centos7",
        :eth1 => "192.168.205.10",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "k8s-node-1",
        :type => "node",
        :box => "geerlingguy/centos7",
        :eth1 => "192.168.205.11",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "k8s-node-2",
        :type => "node",
        :box => "geerlingguy/centos7",
        :eth1 => "192.168.205.12",
        :mem => "2048",
        :cpu => "2"
    }
]

# This script to install k8s using kubeadm will get executed after a box is provisioned
$configureBox = <<-SCRIPT
    echo "This is box"
    
    #cp /tmp/sysctl /etc/sysctl.d/50-kubernetes
    #cp /tmp/modules /etc/modules-load.d/50-kubernetes

    echo #modules required for rke > /etc/
    echo br_netfilter >>  /etc/modules-load.d/50-kubernetes.conf
    echo ip6_udp_tunnel  >>  /etc/modules-load.d/50-kubernetes.conf
    echo ip_set >> /etc/modules-load.d/50-kubernetes.conf
    echo ip_set_hash_ip >> /etc/modules-load.d/50-kubernetes.conf
    echo ip_set_hash_net >> /etc/modules-load.d/50-kubernetes.conf
    echo iptable_filter >> /etc/modules-load.d/50-kubernetes.conf
    echo iptable_nat >> /etc/modules-load.d/50-kubernetes.conf
    echo iptable_mangle >> /etc/modules-load.d/50-kubernetes.conf
    echo iptable_raw >> /etc/modules-load.d/50-kubernetes.conf
    echo nf_conntrack_netlink >> /etc/modules-load.d/50-kubernetes.conf
    echo nf_conntrack >> /etc/modules-load.d/50-kubernetes.conf
    echo nf_conntrack_ipv4 >> /etc/modules-load.d/50-kubernetes.conf
    echo nf_defrag_ipv4 >> /etc/modules-load.d/50-kubernetes.conf
    echo nf_nat >> /etc/modules-load.d/50-kubernetes.conf
    echo nf_nat_ipv4 >> /etc/modules-load.d/50-kubernetes.conf
    echo nf_nat_masquerade_ipv4 >> /etc/modules-load.d/50-kubernetes.conf
    echo nfnetlink >> /etc/modules-load.d/50-kubernetes.conf
    echo udp_tunnel >> /etc/modules-load.d/50-kubernetes.conf
    echo veth >> /etc/modules-load.d/50-kubernetes.conf
    echo vxlan >> /etc/modules-load.d/50-kubernetes.conf
    echo x_tables >> /etc/modules-load.d/50-kubernetes.conf
    echo xt_addrtype >> /etc/modules-load.d/50-kubernetes.conf
    echo xt_conntrack >> /etc/modules-load.d/50-kubernetes.conf
    echo xt_comment >> /etc/modules-load.d/50-kubernetes.conf
    echo xt_mark >> /etc/modules-load.d/50-kubernetes.conf
    echo xt_multiport >> /etc/modules-load.d/50-kubernetes.conf
    echo xt_nat >> /etc/modules-load.d/50-kubernetes.conf
    echo xt_recent >> /etc/modules-load.d/50-kubernetes.conf
    echo xt_set >> /etc/modules-load.d/50-kubernetes.conf
    echo xt_statistic >> /etc/modules-load.d/50-kubernetes.conf
    echo xt_tcpudp >> /etc/modules-load.d/50-kubernetes.conf
    echo net.bridge.bridge-nf-call-iptables=1 >> /etc/sysctl.d/50-kubernetes.conf


    modprobe br_netfilter
    modprobe ip6_udp_tunnel
    modprobe ip_set
    modprobe ip_set_hash_ip
    modprobe ip_set_hash_net
    modprobe iptable_filter
    modprobe iptable_nat
    modprobe iptable_mangle
    modprobe iptable_raw
    modprobe nf_conntrack_netlink
    modprobe nf_conntrack
    modprobe nf_conntrack_ipv4
    modprobe nf_defrag_ipv4
    modprobe nf_nat
    modprobe nf_nat_ipv4
    modprobe nf_nat_masquerade_ipv4
    modprobe nfnetlink
    modprobe udp_tunnel
    modprobe veth
    modprobe vxlan
    modprobe x_tables
    modprobe xt_addrtype
    modprobe xt_conntrack
    modprobe xt_comment
    modprobe xt_mark
    modprobe xt_multiport
    modprobe xt_nat
    modprobe xt_recent
    modprobe xt_set
    modprobe xt_statistic
    modprobe xt_tcpudp
    sysctl net.bridge.bridge-nf-call-iptables=1 
    
    #groupadd dockerroot
    #echo { \\"group\\": \\"dockerroot\\" } > /etc/docker/daemon.json
    #chown :dockerroot /var/run/docker.sock
    #echo 'DOCKER_OPTS="--group dockerroot"' > /etc/default/docker
    yum install -y docker
    groupadd docker
    systemctl enable docker
    systemctl start docker

    usermod -aG docker vagrant
    #echo 'OPTIONS="$OPTIONS --group dockerroot"' >> /etc/sysconfig/docker
    #OPTIONS="--group dockerroot" systemctl start docker
SCRIPT

$configureMaster = <<-SCRIPT
    echo "This is master"
    sudo -u vagrant curl -sL -o /home/vagrant/rke https://github.com/rancher/rke/releases/download/v0.1.13/rke_linux-amd64
    chmod +x /home/vagrant/rke
    sudo -u vagrant sh -c 'yes | ssh-keygen -f /vagrant/kubekey -N ""'
    sudo -u vagrant sh -c 'echo `cat /vagrant/kubekey.pub` >> ~/.ssh/authorized_keys'
SCRIPT

$configureNode = <<-SCRIPT
    echo "This is worker"
    sudo -u vagrant sh -c 'echo `cat /vagrant/kubekey.pub` >> ~/.ssh/authorized_keys'
SCRIPT

Vagrant.configure("2") do |config|

    servers.each do |opts|
        config.vm.define opts[:name] do |config|

            config.vm.box = opts[:box]
            config.vm.box_version = opts[:box_version]
            config.vm.hostname = opts[:name]
            config.vm.network :private_network, ip: opts[:eth1]
            config.vm.provision "file", source: Dir.getwd + "/sysctl", destination: "/tmp/sysctl"
            config.vm.provision "file", source: Dir.getwd + "/modules", destination: "/tmp/modules"
            config.vm.synced_folder ".", "/vagrant"

            #config.vm.network :private_network
            #config.vm.network :private_network, type: "dhcp"

            config.vm.provider "virtualbox" do |v|
                v.name = opts[:name]
             	  v.customize ["modifyvm", :id, "--groups", "/vagrant-kubernetest"]
                v.customize ["modifyvm", :id, "--memory", opts[:mem]]
                v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]
            end

            # we cannot use this because we can't install the docker version we want - https://github.com/hashicorp/vagrant/issues/4871
            #config.vm.provision "docker"

            config.vm.provision "shell", inline: $configureBox
            if opts[:type] == "master"
                config.vm.provision "shell", inline: $configureMaster
            else
                config.vm.provision "shell", inline: $configureNode
            end

        end

    end

end 
