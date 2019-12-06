$script_mysql = <<-SCRIPT
  apt-get update && \
  apt-get install -y mysql-server-5.7 && \
  mysql -e "create user 'phpuser'@'%' identified by 'pass';"
SCRIPT

Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/bionic64"
    config.vm.provider "virtualbox" do |v|
    v.memory = 512
    v.cpus = 1

    if Vagrant.has_plugin?("vagrant-proxyconf")
        config.proxy.http     = "http://10.0.2.2:3128"  
        config.proxy.https    = "http://10.0.2.2:3128"  
        config.proxy.no_proxy = "localhost,127.0.0.1"  
    end
end
    # config.vm.define "mysqldb" do |mysql|
        
    #     mysql.vm.network "public_network", ip: "192.168.174.129"
    #     mysql.vm.synced_folder "./configs", "/configs"
    #     mysql.vm.synced_folder ".", "/vagrant", disable: true
    #     mysql.vm.provision "shell", inline: "cat /configs/mykey.pub >> .ssh/authorized_keys"
    #     mysql.vm.provision "shell", inline: $script_mysql
    #     mysql.vm.provision "shell", inline: "cat /configs/mysqld.cnf > /etc/mysql/mysql.conf.d/mysqld.cnf"
    #     mysql.vm.provision "shell", inline: "service mysql restart"
    # end


    config.vm.define "phpweb" do |phpweb|
        phpweb.vm.network "forwarded_port", guest: 8888, host: 8888
        phpweb.vm.network "public_network", ip: "192.168.174.130"
        phpweb.vm.provision "shell", inline: "apt-get update && apt-get install -y puppet"

        phpweb.vm.provision "puppet" do |puppet|
            puppet.manifests_path = "./configs/manifest"
            puppet.manifest_file = "phpweb.pp"
        end

    end

    config.vm.define "mysqlserver" do |mysqlserver|
        mysqlserver.vm.network "public_network", ip: "192.168.174.131"
        mysqlserver.vm.provision "shell", inline: "cat /vagrant/configs/mykey.pub >> .ssh/authorized_keys"
    end
    
    config.vm.define "ansible" do |ansible|
        ansible.vm.network "public_network", ip: "192.168.174.132"
        ansible.vm.provision "shell", inline: "cp /vagrant/mykey /home/vagrant && \ 
                                               chmod 600 /home/vagrant/mykey && \
                                               chown vagrant:vagrant /home/vagrant/mykey"

        ansible.vm.provision "shell", 
            inline: "apt-get update && \
            apt-get install -y software-properties-common && \
            apt-add-repository --yes --update ppa:ansible/ansible && \
            apt-get install -y ansible"
        
        ansible.vm.provision "shell", 
            inline: "ansible-playbook -i /vagrant/configs/ansible/hosts \
                    /vagrant/configs/ansible/playbook.yml"
    end

    config.vm.define "memcached" do |memcached|
        memcached.vm.box = "centos/7"
        memcached.vm.provider "virtualbox" do |vb|
            vb.memory = 512
            vb.cpus = 1
            vb.name = "centos7_memcache"
        end
    end

end
