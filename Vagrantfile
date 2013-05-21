box_default     = "centos-64-x64-vbox4210"
domain_default  = "vagrant.deveco.yb.int"

nodes       = {
    :web01 => { :ip => "172.16.201.10" },
    :web02 => { :ip => "172.16.201.11" },

    :app01 => { :ip => "172.16.201.20" },
    :app02 => { :ip => "172.16.201.21" },

    :sql01 => { :ip => "172.16.201.30" },
    :sql02 => { :ip => "172.16.201.31" },
}

Vagrant.require_plugin "vagrant-vbguest"

Vagrant::configure("2") do |config|
    nodes.each do |node_name, node_opts|
        config.vm.define node_name do |c|

            c.vm.box = node_opts[:box] || box_default
            domain = node_opts[:domain] || domain_default
            fqdn = "#{node_name}.#{domain}"

            c.vm.hostname = fqdn
            #c.vbguest.auto_update = false
            c.ssh.forward_agent = true

            # virtualbox provider host setting modifications
            c.vm.provider :virtualbox do |vb|
                modifyvm_args = ['modifyvm', :id]
                modifyvm_args << '--name' << fqdn
                
                # workaround for https://github.com/mitchellh/vagrant/issues/516
                modifyvm_args << '--nictype1' << 'Am79C973'

                # Isolate guests from host networking.
                modifyvm_args << '--natdnsproxy1' << 'on'
                modifyvm_args << '--natdnshostresolver1' << 'on'

                unless node_opts[:memory].nil?
                    modifyvm_args << '--memory' << node_opts[:memory].to_s
                end

                vb.customize(modifyvm_args)
            end

            if node_opts[:network] == :public_network
                c.vm.network :public_network
            elsif node_opts[:ip]
                c.vm.network :private_network, ip: node_opts[:ip]
            end

            unless node_opts[:puppet] == false
                c.vm.synced_folder "puppet/", "/tmp/vagrant-puppet/", id: "puppet-data"
                c.vm.provision :puppet do |puppet|
                    puppet.manifest_file = "site.pp"
                    puppet.manifests_path = "puppet/manifests"
                    puppet.module_path = "puppet/modules"
                end
            end
        end
    end
end
