box_default     = "centos64-puppet32"
domain_default  = "vagrant.deveco.yb.int"

nodes       = {
    :eco51 => { :ip => "172.16.201.51" },
    :eco52 => { :ip => "172.16.201.52" },
    :eco53 => { :ip => "172.16.201.53" },
    :eco54 => { :ip => "172.16.201.54" },
    :eco55 => { :ip => "172.16.201.55" },
    :eco56 => { :ip => "172.16.201.56" },
    :eco57 => { :ip => "172.16.201.57" },
    :eco58 => { :ip => "172.16.201.58" },
    :eco59 => { :ip => "172.16.201.59" },
    :eco60 => { :ip => "172.16.201.60" },
    :eco61 => { :ip => "172.16.201.61" },
    :eco62 => { :ip => "172.16.201.62" },
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
