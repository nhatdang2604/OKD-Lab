Vagrant.configure("2") do |config|

  # Make things don't update
  config.vm.box_check_update = false

  # Should change for each installation
  base_storage_path         = "F:/Vagrant/VMs"
  base_data_path            = "./data"
  bridged_network_interface = "Ethernet adapter Ethernet"
  bridged_network_name      = "eth1"
  dns_ip                    = "8.8.8.8"

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

    'dns' => {
      'os'   => {
        'box'     => 'centos/stream9',
        'version' => '20250210.0'
      },
      'memory'  => "1024", # MB
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
    }
  }

  # Network settings
  vms = [
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

    {
      'name'    => 'dns',
      'os'      => common_settings['dns']['os'],
      'memory'  => common_settings['dns']['memory'],
      'cpus'    => common_settings['dns']['cpus'],
      'storage' => common_settings['dns']['storage'],
      'network' => {
        'name'           => common_settings['dns']['network']['name'],
        'type'           => common_settings['dns']['network']['type'],
        'interface'      => common_settings['okd']['network']['interface'],
        'interface_name' => common_settings['okd']['network']['interface_name'],
        'ip'             => '192.168.1.104',
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

      # Setup sync folder
      # host_dir  = "#{base_data_path}/#{vm_name}/"
      # guest_dir = "/home/vagrant/#{vm_name}/"
      # node.vm.synced_folder host_dir, guest_dir

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
          ipv4.addresses #{vm_network['ip']}/24 ipv4.gateway #{vm_network['gateway']} ipv4.dns #{dns_ip} ipv4.method manual
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
          echo "Start to copy config file"  
          
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
          
        SHELL

      end
    end
  end
end

  