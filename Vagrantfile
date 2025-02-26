Vagrant.configure("2") do |config|

  # Make things don't update
  config.vm.box_check_update = false

  # Should change for each installation
  base_storage_path         = "F:/Vagrant/VMs"
  base_data_path            = "./data"
  bridged_network_interface = "Ethernet adapter Ethernet"
  bridged_network_name      = "eth1"

  # Define the box URL for OSs
  common_settings = {
    'okd' => {
      'os' => {
        'box'     => "fedora/38-cloud-base",
        'version' => "38.20230413.1"
      },
      'memory'  => "4096", # MB
      'cpus'    => 2, #vCPUs
      'storage' => {
        'path' => base_storage_path,
        'size' => 60 * 1024 # MB
      },
      'network' => {
        'name'           => 'public_network',
        'type'           => 'bridged',
        'interface'      => bridged_network_interface,
        'interface_name' => bridged_network_name,
      }
    },

    'lb' => {
      'os'   => {
        'box'     => 'centos/stream9',
        'version' => '20250210.0'
      },
      'memory'  => "2048", # MB
      'cpus'    => 2, #vCPUs
      'storage' => {
        'path' => base_storage_path,
        'size' => 30 * 1024 # MB
      },
      'network' => {
        'name'           => 'public_network',
        'type'           => 'bridged',
        'interface'      => bridged_network_interface,
        'interface_name' => bridged_network_name,
      }
    },

    'dns' => {
      'os'   => {
        'box'     => 'centos/stream9',
        'version' => '20250210.0'
      },
      'memory'  => "4096", # MB
      'cpus'    => 2, #vCPUs
      'storage' => {
        'path' => base_storage_path,
        'size' => 30 * 1024 # MB
      },
      'network' => {
        'name'           => 'public_network',
        'type'           => 'bridged',
        'interface'      => bridged_network_interface,
        'interface_name' => bridged_network_name,
        'ip'             => '192.168.1.104',
      }
    }
  }

  # VM settings
  # The order of VMs is important
  vms = [
    {
      'name'    => 'dns',
      'os'      => common_settings['dns']['os'],
      'memory'  => common_settings['dns']['memory'],
      'cpus'    => common_settings['dns']['cpus'],
      'storage' => common_settings['dns']['storage'],
      'network' => {
        'name'           => common_settings['dns']['network']['name'],
        'type'           => common_settings['dns']['network']['type'],
        'interface'      => common_settings['dns']['network']['interface'],
        'interface_name' => common_settings['dns']['network']['interface_name'],
        'ip'             => common_settings['dns']['network']['ip'],
        'gateway'        => '192.168.1.1',
      },
    },

    {
      'name'    => 'load-balancer',
      'os'      => common_settings['lb']['os'],
      'memory'  => common_settings['lb']['memory'],
      'cpus'    => common_settings['lb']['cpus'],
      'storage' => common_settings['lb']['storage'],
      'network' => {
        'name'           => common_settings['lb']['network']['name'],
        'type'           => common_settings['lb']['network']['type'],
        'interface'      => common_settings['lb']['network']['interface'],
        'interface_name' => common_settings['lb']['network']['interface_name'],
        'ip'             => '192.168.1.105',
        'gateway'        => '192.168.1.1',
      },
    },

    {
      'name'    => 'bootstrap',
      'os'      => common_settings['okd']['os'],
      'memory'  => common_settings['okd']['memory'],
      'cpus'    => common_settings['okd']['cpus'],
      'storage' => common_settings['okd']['storage'],
      'network' => {
        'name'           => common_settings['okd']['network']['name'],
        'type'           => common_settings['okd']['network']['type'],
        'interface'      => common_settings['okd']['network']['interface'],
        'interface_name' => common_settings['okd']['network']['interface_name'],
        'ip'             => '192.168.1.101',
        'gateway'        => '192.168.1.1',
      },
    },

    {
      'name'    => 'controller',
      'os'      => common_settings['okd']['os'],
      'memory'  => common_settings['okd']['memory'],
      'cpus'    => common_settings['okd']['cpus'],
      'storage' => common_settings['okd']['storage'],
      'network' => {
        'name'           => common_settings['okd']['network']['name'],
        'type'           => common_settings['okd']['network']['type'],
        'interface'      => common_settings['okd']['network']['interface'],
        'interface_name' => common_settings['okd']['network']['interface_name'],
        'ip'             => '192.168.1.102',
        'gateway'        => '192.168.1.1',
      },
    },

    {
      'name'    => 'compute',
      'os'      => common_settings['okd']['os'],
      'memory'  => common_settings['okd']['memory'],
      'cpus'    => common_settings['okd']['cpus'],
      'storage' => common_settings['okd']['storage'],
      'network' => {
        'name'           => common_settings['okd']['network']['name'],
        'type'           => common_settings['okd']['network']['type'],
        'interface'      => common_settings['okd']['network']['interface'],
        'interface_name' => common_settings['okd']['network']['interface_name'],
        'ip'             => '192.168.1.103',
        'gateway'        => '192.168.1.1',
      },
    },
  ]

  # Setup for each VM
  vms.each do |vm|
    config.vm.define vm['name'] do |node|
      
      # Setup bio
      node.vm.box = vm['os']['box']
      node.vm.box_version = vm['os']['version']
      node.vm.hostname = vm['name']

      # Setup workspace
      vm_name = vm['name']
      vm_storage_path = "#{vm['storage']['path']}/#{vm_name}"
      unless Dir.exist?(vm_storage_path)
        Dir.mkdir(vm_storage_path)
      end

      # Setup network with Static ip on bridged network interface
      vm_network = vm['network']
      node.vm.network vm_network['name'], bridge: vm_network['interface'], type: 'static', ip: vm_network['ip']  
      node.vm.provision "shell", inline: <<-SHELL
        echo "Attempt to set static ip for #{vm_name}"
        
        set -x
        echo "Debug mode on"
        
        CONNECTION_NAME=$(nmcli -t -f NAME,DEVICE connection show | grep "#{vm_network['interface_name']}" | cut -d: -f1)
        echo $CONNECTION_NAME

        sudo nmcli connection modify "$CONNECTION_NAME" \
          ipv4.addresses #{vm_network['ip']}/24 ipv4.gateway #{vm_network['gateway']} ipv4.dns #{common_settings['okd']['network']['ip']} ipv4.method manual
        echo "Set network manually done"

        sudo nmcli connection down "$CONNECTION_NAME" && sudo nmcli connection up "$CONNECTION_NAME"
        echo "Restart network done"

      SHELL
      
      # VMs resources
      node.vm.provider "virtualbox" do |vb|
        vb.memory = vm['memory']
        vb.cpus   = vm['cpus'] 
        vb.name   = vm['name']

        # Setup storage
        vdi_path = "#{vm_storage_path}/#{vm_name}.vdi"
        unless File.exist?(vdi_path)
          vb.customize ["createhd", "--filename", vdi_path, "--size", vm['storage']['size']]
          vb.customize ["storagectl", vm_name, "--name", "SATA", "--add", "sata", "--controller", "IntelAhci"]
          vb.customize ["storageattach", vm_name, "--storagectl", "SATA", "--port", "1", "--device", "0", "--type", "hdd", "--medium", vdi_path]
        end
      end

      # DNS setup
      if "dns" == vm_name
        
        # Copy config file
        node.vm.provision "shell", inline: <<-SHELL
          echo "Start to init DNS/Service Node"  
          
          # Debug mode on
          set -x
          echo "Debug mode on"

          # Setup config file from host machine to VM
          sudo mkdir /etc/named/
          sudo mkdir /etc/named/zones
          sudo cp /vagrant/data/dns/etc/named.conf /etc/named.conf
          sudo cp /vagrant/data/dns/etc/named/named.conf.local /etc/named/named.conf.local
          sudo cp -r /vagrant/data/dns/etc/named/zones/ /etc/named/
          echo "Copy config files done"

          # Setup FirewallD
          sudo systemctl start firewalld
          sudo firewall-cmd --permanent --add-port=53/udp
          sudo firewall-cmd --reload
          sudo systemctl status firewalld

          # Setup DNS server
          sudo dnf -y install bind bind-utils
          sudo systemctl enable named
          sudo systemctl stop named
          sudo systemctl start named
          sudo systemctl status named
          echo "Restart named service done"

          # Config the current DNS server of the DNS machine become localhost
          PRIMARY_CONNECTION_NAME=$(nmcli -t -f NAME,DEVICE connection show | grep "eth0" | cut -d: -f1)
          SECONDARY_CONNECTION_NAME=$(nmcli -t -f NAME,DEVICE connection show | grep "eth1" | cut -d: -f1)
          sudo nmcli connection modify "$PRIMARY_CONNECTION_NAME" ipv4.dns "127.0.0.1"
          sudo nmcli connection modify "$SECONDARY_CONNECTION_NAME" ipv4.dns "127.0.0.1"
          sudo nmcli connection down "$PRIMARY_CONNECTION_NAME" && sudo nmcli connection up "$PRIMARY_CONNECTION_NAME"
          sudo nmcli connection down "$SECONDARY_CONNECTION_NAME" && sudo nmcli connection up "$SECONDARY_CONNECTION_NAME"
          echo "Setup dns done"          

          # Test DNS resolve
          nslookup okd4-services.okd.local
          nslookup okd4-bootstrap.cloud.okd.local
          nslookup okd4-control.cloud.okd.local
          nslookup okd4-compute.cloud.okd.local
          nslookup etcd.cloud.okd.local
          nslookup console-openshift-console.apps.cloud.okd.local
          nslookup oauth-openshift.apps.cloud.okd.local
          nslookup google.com

          # Setup httpd
          sudo dnf install -y httpd
          sudo sed -i 's/Listen 80/Listen 8080/' /etc/httpd/conf/httpd.conf 
          sudo setsebool -P httpd_read_user_content 1
          sudo firewall-cmd --permanent --add-port=8080/tcp
          sudo firewall-cmd --reload
          sudo systemctl enable httpd
          sudo systemctl start httpd

          # Setup OKD Client
          sudo dnf install -y wget
          mkdir /home/vagrant/softs
          wget -P /home/vagrant/softs https://github.com/okd-project/okd/releases/download/4.15.0-0.okd-2024-03-10-010116/openshift-client-linux-4.15.0-0.okd-2024-03-10-010116.tar.gz
          wget -P /home/vagrant/softs https://github.com/okd-project/okd/releases/download/4.15.0-0.okd-2024-03-10-010116/openshift-install-linux-4.15.0-0.okd-2024-03-10-010116.tar.gz
          tar -xvzf /home/vagrant/softs/openshift-client-linux-4.15.0-0.okd-2024-03-10-010116.tar.gz -C /home/vagrant/softs
          tar -xvzf /home/vagrant/softs/openshift-install-linux-4.15.0-0.okd-2024-03-10-010116.tar.gz -C /home/vagrant/softs
          sudo mv /home/vagrant/softs/kubectl /home/vagrant/softs/oc /home/vagrant/softs/openshift-install /usr/local/bin

          # Setup OKD config
          mkdir /home/vagrant/install_dir
          ssh-keygen -t rsa -b 2048 -f /home/vagrant/.ssh/id_rsa -N ""
          sudo cp /vagrant/data/dns/install_dir/install-config.yaml /home/vagrant/install_dir/install-config.yaml
          sudo sed -i "s|sshKey: 'replace_your_ssh_key_here'|sshKey: '$(cat /home/vagrant/.ssh/id_rsa.pub)'|" /home/vagrant/install_dir/install-config.yaml
          sudo cp /home/vagrant/install_dir/install-config.yaml /home/vagrant/install_dir/install-config.yaml.bak # Backup, because after generating Manifests, the old install-config.yaml will be removed
        
          # Installing OKD
          /usr/local/bin/openshift-install create manifests --dir=/home/vagrant/install_dir/
          sudo sed -i "s|mastersSchedulable: true|mastersSchedulable: False|" /home/vagrant/install_dir/manifests/cluster-scheduler-02-config.yml
          /usr/local/bin/openshift-install create ignition-configs --dir=/home/vagrant/install_dir/

          # Copy OKD config to webserver
          sudo mkdir /var/www/html/okd4
          sudo cp -R /home/vagrant/install_dir/* /var/www/html/okd4/
          sudo chown -R apache: /var/www/html/
          sudo chmod -R 755 /var/www/html/
          echo "export KUBECONFIG=/home/vagrant/install_dir/auth/kubeconfig" >> /home/vagrant/.bash_profile
          . /home/vagrant/.bash_profile

          # Install JQ for node aprovement
          wget -P /home/vagrant/softs -O /home/vagrant/softs/jq https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64
          chmod +x /home/vagrant/softs/jq
          sudo mv /home/vagrant/softs/jq /usr/local/bin/
          /usr/local/bin/jq --version 

        SHELL
      end

      # Load Balancer setup
      if "load-balancer" == vm_name
        
        node.vm.provision "shell", inline: <<-SHELL
          echo "Start to init Load Balancer Node"   
          
          # Debug mode on
          set -x
          echo "Debug mode on"

          # Setup FirewallD
          sudo systemctl start firewalld
          sudo systemctl status firewalld
          sudo firewall-cmd --permanent --add-port=6443/tcp
          sudo firewall-cmd --permanent --add-port=22623/tcp
          sudo firewall-cmd --permanent --add-service=http
          sudo firewall-cmd --permanent --add-service=https
          sudo firewall-cmd --permanent --add-port=9000/tcp
          sudo firewall-cmd --reload

          # Setup HA Proxy
          sudo dnf install haproxy -y
          sudo cp /vagrant/data/load-balancer/etc/haproxy.cfg /etc/haproxy/haproxy.cfg
          sudo setsebool -P haproxy_connect_any 1
          sudo systemctl enable haproxy
          sudo systemctl start haproxy
          sudo systemctl status haproxy

        SHELL
      end

      # Bootstrap setup
      if "bootstrap" == vm_name
        node.vm.provision "shell", inline: <<-SHELL
          echo "Start to init Bootstrap Node"   
          
          # Debug mode on
          set -x
          echo "Debug mode on"

          # Install Fedora CoreOS
          sudo dnf install coreos-installer -y
          OKD_SERVICE_URL=#{common_settings['dns']['network']['ip']}
          sudo coreos-installer install --ignition-url http://$OKD_SERVICE_URL:8080/okd4/bootstrap.ign /dev/sda --insecure-ignition --copy-network
          sudo reboot

        SHELL
      end

    end
  end
end

  