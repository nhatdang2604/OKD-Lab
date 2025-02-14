Vagrant.configure("2") do |config|

  # Make things don't update
  config.vm.box_check_update = false

  # Should change for each installation
  base_storage_path         = "F:/Vagrant/VMs"
  bridged_network_interface = "Ethernet"

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
        'name'      => 'public_network',
        'type'      => 'bridged',
        'interface' => bridged_network_interface
      }
    },

    'dns' => {
      'os'   => {
        'box'     => 'fedora/38-cloud-base',
        'version' => '38.20230413.1'
      },
      'memory'  => "1024", # MB
      'cpus'    => 2, #vCPUs
      'storage' => {
        'path' => base_storage_path,
        'size' => 30 * 1024 # MB
      },
      'network' => {
        'name'      => 'public_network',
        'type'      => 'bridged',
        'interface' => bridged_network_interface
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
        'name'      => common_settings['okd']['network']['name'],
        'type'      => common_settings['okd']['network']['type'],
        'interface' => common_settings['okd']['network']['interface'],
        'ip'        => '192.168.56.101',
      },
    },

    {
      'name'    => 'controller',
      'os'      => common_settings['okd']['os'],
      'memory'  => common_settings['okd']['memory'],
      'cpus'    => common_settings['okd']['cpus'],
      'storage' => common_settings['okd']['storage'],
      'network' => {
        'name'      => common_settings['okd']['network']['name'],
        'type'      => common_settings['okd']['network']['type'],
        'interface' => common_settings['okd']['network']['interface'],
        'ip'        => '192.168.56.102',
      },
    },

    {
      'name'    => 'compute',
      'os'      => common_settings['okd']['os'],
      'memory'  => common_settings['okd']['memory'],
      'cpus'    => common_settings['okd']['cpus'],
      'storage' => common_settings['okd']['storage'],
      'network' => {
        'name'      => common_settings['okd']['network']['name'],
        'type'      => common_settings['okd']['network']['type'],
        'interface' => common_settings['okd']['network']['interface'],
        'ip'        => '192.168.56.103',
      },
    },

    {
      'name'    => 'dns',
      'os'      => common_settings['dns']['os'],
      'memory'  => common_settings['dns']['memory'],
      'cpus'    => common_settings['dns']['cpus'],
      'storage' => common_settings['dns']['storage'],
      'network' => {
        'name'      => common_settings['dns']['network']['name'],
        'type'      => common_settings['dns']['network']['type'],
        'interface' => common_settings['okd']['network']['interface'],
        'ip'        => '192.168.56.104',
      },
    },
  ]

  vms.each do |vm|
    config.vm.define vm['name'] do |node|
      node.vm.box = vm['os']['box']
      node.vm.box_version = vm['os']['version']
      node.vm.hostname = vm['name']

      # VMs resources
      node.vm.provider "virtualbox" do |vb|
        vb.memory = vm['memory']
        vb.cpus   = vm['cpus'] 
        vb.name   = vm['name']

        # Setup workspace
        vm_name = vm['name']
        vm_storage_path = "#{vm['storage']['path']}/#{vm_name}"
        unless Dir.exist?(vm_storage_path)
          Dir.mkdir(vm_storage_path)
        end

        # Setup storage
        vdi_path = "#{vm_storage_path}/#{vm_name}.vdi"
        unless File.exist?(vdi_path)
          vb.customize ["createhd", "--filename", vdi_path, "--size", vm['storage']['size']]
        end
        vb.customize ["storagectl", vm_name, "--name", "SATA", "--add", "sata", "--controller", "IntelAhci"]
        vb.customize ["storageattach", vm_name, "--storagectl", "SATA", "--port", "1", "--device", "0", "--type", "hdd", "--medium", vdi_path]

      end

      # Static IP and networking setup
      vm_network = vm['network']
      node.vm.network vm_network['name'], bridge: vm_network['interface'], type: 'static', ip: vm_network['ip']
      
    end
  end
end
