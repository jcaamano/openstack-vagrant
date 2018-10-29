require 'yaml'

def escape(s)
    "\"" + s.gsub("\\", "\\\\").gsub("\"", "\\\"") + "\""
end

directory = YAML.load_file("directory.conf.yml")

username = "root"

Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "data", "/vagrant_data", type: "rsync"
  config.vm.synced_folder 'common', '/vagrant_common', type: "rsync"

  # Setup a private network as public, with default devstack address
  config.vm.network "private_network",
      libvirt__host_ip: "192.168.42.129",
      libvirt__network_address: "192.168.42.0",
      libvirt__dhcp_stop: "192.168.42.10",
      libvirt__dhcp_enabled: true,
      type: "dhcp"

  ansible_host_vars = {}
  ansible_hosts = []
  ansible_config = nil
  directory['machines'].each do |machine|
    if (ARGV.index 'up' or ARGV.index 'provision') and ARGV.index machine['name']
      ansible_hosts.push(machine['name'])
      ansible_config = machine['name']
    end
    if ansible_hosts.empty?
      ansible_config = machine['name']
    end
  end

  directory['machines'].each do |machine|
    config.vm.define machine['name'] do |config|

      # There seems to be compatibility problems between gnu & bsd netcat.
      # Workaround for SUSE hosts (bsd netcat).
      config.ssh.proxy_command = "ssh #{machine['hypervisor']['name']} -l #{machine['hypervisor']['username']} nc -N %h %p"

      config.vm.hostname = machine['name']
      config.vm.box = machine['box']
      config.vm.provider :libvirt do |libvirt|
        libvirt.host = machine['hypervisor']['name']
        libvirt.connect_via_ssh = true
        libvirt.username = machine['hypervisor']['username']
        libvirt.memory = machine['memory']
        libvirt.cpus = machine['vcpus']
        libvirt.cpu_mode = 'host-passthrough'
      end

      projects = machine['projects']
      ansible_host_vars[machine['name']] = {
        ansible_ssh_common_args: escape("-o ProxyCommand=\"ssh '#{machine['hypervisor']['name']}' -l '#{machine['hypervisor']['username']}' -i '/#{ENV['HOME']}/.ssh/id_rsa' nc %h %p\""),
        local_conf_file: machine['local_conf_file'],
        projects: "'#{projects.to_json}'",
        devstack_version: machine['devstack_version']
      }

      if ansible_config == machine['name']
        config.vm.provision "ansible" do |ansible|
          ansible.playbook = "devstack.yml"
          #ansible.raw_arguments = "-vvv"
          ansible.host_vars = ansible_host_vars
          ansible.limit = ansible_hosts.empty? ? 'all' : ansible_hosts
        end
      end
    end
  end

end
