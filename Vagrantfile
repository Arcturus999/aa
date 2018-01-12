

require 'securerandom'

require 'yaml'
aws_config = YAML::load_file("#{File.dirname(File.expand_path(__FILE__))}/aws-config.yml")


Vagrant.configure(2) do |config|
  
  all_created_cassandra_nodes = []
  datacenter_and_nodes = {}
  
  aws_config["datacenters"].each do |datacenter_info|
    
    
    datacenter_and_nodes[datacenter_info["region"][0..-3]] = datacenter_info["number_of_nodes"]
    
    datacenter_info["number_of_nodes"].times do |node_number|
      
      node_name = ( aws_config["node_name_prefix"] ? aws_config["node_name_prefix"] + "-" : "" ) + "cassandra-" + datacenter_info["region"] + "-" + node_number.to_s
      
      all_created_cassandra_nodes << node_name
      
      config.vm.define(node_name) do |node|  
      
          node.vm.box = "ubuntu/trusty64"
          node.vm.hostname = node_name
          
          
          node.vm.provider :virtualbox do |v|
            v.name = node_name
          end

          node.vm.provider :aws do |aws, override|
            aws.access_key_id = aws_config["key"]
            aws.secret_access_key = aws_config["secret"]
            
            aws.keypair_name = aws_config["keyname"]
        
            aws.ami = datacenter_info["ami"]
            aws.region = datacenter_info["region"]
            aws.instance_type = datacenter_info["instance_type"]
            aws.security_groups = ["default", datacenter_info["security_group"]]

            override.vm.box = "dummy"
            override.ssh.username = "ubuntu"
            override.ssh.private_key_path = aws_config["keypath"]
          end
        end
    
    end
  end
  
 
  
  opscenter_info = aws_config["opscenter"]
  
  config.vm.define(opscenter_info["node_name"]) do |node|  
  
    node.vm.box = "ubuntu/trusty64"
    node.vm.hostname = opscenter_info["node_name"]
   
    
    node.vm.provider :virtualbox do |v|
      v.name = opscenter_info["node_name"]
    end

    node.vm.provider :aws do |aws, override|
      aws.access_key_id = aws_config["key"]
      aws.secret_access_key = aws_config["secret"]
      
      aws.keypair_name = aws_config["keyname"]
  
      aws.ami = opscenter_info["ami"]
      aws.region = opscenter_info["region"]
      aws.instance_type = opscenter_info["instance_type"]
      aws.security_groups = ["default", opscenter_info["security_group"]]

      override.vm.box = "dummy"
      override.ssh.username = "ubuntu"
      override.ssh.private_key_path = aws_config["keypath"]
    end
    
    
    # Provisionning for the nodes using ansible !! remember to install java 7
    #tho using ansible 1.9 in my machine with sudo pip install 'ansible==1.9.1' along with vagrant 2.0.1 note to myself not to install last version via sudo apt-get as it containes errors
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'ansible/ansible-playbook.yml'
      ansible.limit = 'all'
      ansible.groups = {
        "cassandra_nodes" => all_created_cassandra_nodes,
        "opscenter_node" => [opscenter_info["node_name"]],
        "all:children" => ["cassandra_nodes", "opscenter_node"]
      }
      ansible.extra_vars = {
        "cluster_name" => aws_config["cluster_name"],
        "cluster_username" => aws_config["cluster_username"],
        "cluster_password" => aws_config["cluster_password"],
        "jmx_username" => aws_config["jmx_username"],
        "jmx_password" => aws_config["jmx_password"],
        "datacenter_max_replication_string" => datacenter_and_nodes.map { |k,v| "'#{k}' : #{v}" }.join(", "),
        "random_new_password" => SecureRandom.hex
      }
    end
    
  end
  
  
  config.ssh.insert_key = false

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL
end

