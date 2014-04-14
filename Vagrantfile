# -*- mode: ruby -*-
# vi: set ft=ruby :

## Cassandra cluster settings
server_count = 1
network = '192.168.2.'
first_ip = 10

servers = []
seeds = []
cassandra_tokens = []
(0..server_count-1).each do |i|
  name = 'node' + (i + 1).to_s
  ip = network + (first_ip + i).to_s
  seeds << ip
  servers << {'name' => name,
              'ip' => ip,
              'initial_token' => 2**127 / server_count * i}
end

Vagrant.configure('2') do |config|
  servers.each do |server|
    config.vm.define server['name'] do |config2|
      config2.vm.box = "centos64-x86_64-20131030"
      config2.vm.box_url = "https://github.com/2creatives/vagrant-centos/releases/download/v0.1.0/centos64-x86_64-20131030.box"
      config2.vm.host_name = server['name']
      config2.vm.network "private_network", ip: server['ip']

      config2.vm.provider "virtualbox" do |v|
        v.memory = 2048
      end

      config.omnibus.chef_version = :latest
      config2.vm.provision :chef_solo do |chef|
        chef.log_level = :debug
        # TODO Remove last element of cookbooks_path once done with spark-chef cookbook development
        chef.cookbooks_path = ["vagrant/cookbooks", "vagrant/site-cookbooks", "/Users/brent/cookbooks"]
        chef.add_recipe "cassandra::datastax"
        chef.add_recipe "java"
        chef.add_recipe "spark"
        chef.json = {
          :java => {
            :install_flavor => "oracle",
            :jdk_version => "7",
            :oracle => {
              :accept_oracle_download_terms => true
            }
          },
          :cassandra => {
            :cluster_name => 'My Cluster',
            :initial_token => server['initial_token'],
            :seeds => seeds.join(","),
            :listen_address => server['ip'],
            :rpc_address => server['ip']
          }
        }
      end
    end
  end
end
