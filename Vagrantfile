# -*- mode: ruby -*-
# vi: set ft=ruby :

# Cassandra 3-Node Cluster Configuration
CASSANDRA_VERSION = "4.1"
CLUSTER_NAME = "TestCluster"

# Node configuration
nodes = [
  { :name => "cassandra-node1", :ip => "192.168.8.16", :seed => true },
  { :name => "cassandra-node2", :ip => "192.168.8.17", :seed => false },
  { :name => "cassandra-node3", :ip => "192.168.8.18", :seed => false }
]

# Seed nodes (first node)
SEEDS = "192.168.8.16"

Vagrant.configure("2") do |config|
  # Use Ubuntu 22.04 as base box
  config.vm.box = "ubuntu/jammy64"
  
  # Disable automatic box update checking
  config.vm.box_check_update = false

  nodes.each_with_index do |node, index|
    config.vm.define node[:name] do |node_config|
      node_config.vm.hostname = node[:name]
      node_config.vm.network "private_network", ip: node[:ip]

      # VirtualBox specific settings
      node_config.vm.provider "virtualbox" do |vb|
        vb.name = node[:name]
        vb.memory = 2048
        vb.cpus = 2
        # Disable USB to avoid errors
        vb.customize ["modifyvm", :id, "--usb", "off"]
        vb.customize ["modifyvm", :id, "--usbehci", "off"]
      end

      # Provisioning script for Cassandra installation and configuration
      node_config.vm.provision "shell", inline: <<-SHELL
        set -e

        # Update system
        apt-get update
        apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release

        # Install Java 11 (required for Cassandra)
        apt-get install -y openjdk-11-jdk

        # Add Cassandra repository
        curl -fsSL https://www.apache.org/dist/cassandra/KEYS -o /tmp/cassandra.keys
        gpg --batch --no-tty --dearmor -o /usr/share/keyrings/cassandra-archive-keyring.gpg < /tmp/cassandra.keys
        echo "deb [signed-by=/usr/share/keyrings/cassandra-archive-keyring.gpg] https://debian.cassandra.apache.org 41x main" | tee /etc/apt/sources.list.d/cassandra.sources.list
        rm -f /tmp/cassandra.keys

        # Install Cassandra
        apt-get update
        apt-get install -y cassandra

        # Stop Cassandra to configure
        systemctl stop cassandra
        sleep 5

        # Clear default data
        rm -rf /var/lib/cassandra/data/system/*

        # Configure Cassandra
        CONFIG_FILE="/etc/cassandra/cassandra.yaml"

        # Backup original config
        cp $CONFIG_FILE ${CONFIG_FILE}.bak

        # Update cassandra.yaml configuration
        sed -i "s/cluster_name: 'Test Cluster'/cluster_name: '#{CLUSTER_NAME}'/" $CONFIG_FILE
        sed -i 's/seeds: "127.0.0.1:7000"/seeds: "#{SEEDS}:7000"/' $CONFIG_FILE
        sed -i 's/seeds: "127.0.0.1"/seeds: "#{SEEDS}"/' $CONFIG_FILE
        sed -i "s/listen_address: localhost/listen_address: #{node[:ip]}/" $CONFIG_FILE
        sed -i "s/rpc_address: localhost/rpc_address: #{node[:ip]}/" $CONFIG_FILE
        sed -i "s/# broadcast_address: 1.2.3.4/broadcast_address: #{node[:ip]}/" $CONFIG_FILE
        sed -i "s/# broadcast_rpc_address: 1.2.3.4/broadcast_rpc_address: #{node[:ip]}/" $CONFIG_FILE
        sed -i "s/endpoint_snitch: SimpleSnitch/endpoint_snitch: GossipingPropertyFileSnitch/" $CONFIG_FILE

        # Configure rack and datacenter
        RACKDC_FILE="/etc/cassandra/cassandra-rackdc.properties"
        echo "dc=dc1" > $RACKDC_FILE
        echo "rack=rack1" >> $RACKDC_FILE

        # Start Cassandra
        systemctl start cassandra
        systemctl enable cassandra

        echo "Cassandra node #{node[:name]} (#{node[:ip]}) configured successfully!"
        echo "Wait for the cluster to stabilize before checking status with: nodetool status"
      SHELL
    end
  end
end
