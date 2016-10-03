# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

# Define the cluster
nodes = [
  { :hostname => 'rabbit1', :ip => '192.168.1.11', :box => 'hashicorp/precise32'},
  { :hostname => 'rabbit2', :ip => '192.168.1.12', :box => 'hashicorp/precise32'},
  { :hostname => 'rabbit3', :ip => '192.168.1.13', :box => 'hashicorp/precise32'}
]

rabbitmq_version = '3.3.1'
# ESL needs the 1:{NUMBER} format - use apt-cache to check
erlang_version = '1:17.1'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  #used latest version of chef to help build-essentials work
  config.omnibus.chef_version = :latest

  # add vagrant-cachier
  config.cache.scope = :box

  #Setup hostmanager config to update the host files
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true
  config.vm.provision :hostmanager

  nodes.each do |node|
    config.vm.define node[:hostname] do |node_config|

      # configure the box, hostname and networking
      node_config.vm.box = node[:box]
      node_config.vm.hostname = node[:hostname]
      node_config.vm.network :private_network, ip: node[:ip]

      # configure hostmanager
      node_config.hostmanager.aliases = node[:hostname]

      # use the Chef provisioner to install RabbitMQ
      node_config.vm.provision :chef_solo do |chef|

      #override default chef config
      chef.log_level = :info

      chef.json = {
          'erlang' =>{
              'install_method' => 'esl',
              'esl' => {
                'version' => erlang_version
              }
            },
         'rabbitmq' => {
          #'version' => rabbitmq_version,
          'clustering' => {
            'enable' => true,
            'cluster_name' => 'rabbitcluster',
            'cluster_nodes' => [
              {
                'name' => 'rabbit@rabbit1',
                'type' => 'disc'
              },
              {
                'name' => 'rabbit@rabbit2',
                'type' => 'disc'
              },
              {
                'name' => 'rabbit@rabbit3',
                'type' => 'disc'
              }
            ]
          },
          'erlang_cookie' => "secretSECRETsecret",
          'enabled_users' =>
            [
              {
                'name' => 'remote-guest', 'password' => 'remote-guest', 'tag' => 'administrator', 'rights' =>
                  [{ 'vhost' => nil , 'conf' => '.*', 'write' => '.*', 'read' => '.*' }]
              }
            ],
           'disabled_users' => ['guest'],
           'enabled_plugins' => ['rabbitmq_federation_management'],
           'policies' => {
               'ha-all' => {
                 'pattern' => '^(?!amq\\.).*',
                 'params' => {'ha-mode' => 'all'},
                 'priority' => 0
               }
           },
           'disabled_policies' => []
         }
      }

      chef.add_recipe "rabbitmq::cluster"
      chef.add_recipe "rabbitmq::mgmt_console"
      chef.add_recipe "rabbitmq::user_management"
      chef.add_receipt "rabbitmq::virtualhost_management"
      chef.add_recipe "rabbitmq::policy_management"
      chef.add_recipe "rabbitmq::plugin_management"

      end
    end
  end
end
