# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :mysql => {
        :box_name => "centos/7",
        :ip_addr => '192.168.26.150'
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          box.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "2048"]
          end
          box.vm.provision "shell", inline: <<-SHELL
            mkdir -p ~root/.ssh; cp ~vagrant/.ssh/auth* ~root/.ssh
            sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
            systemctl restart sshd
            yum install epel-release -y
            yum install docker python-pip python-devel gcc -y
            pip install --upgrade pip
            pip install docker-compose
            yum install https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm -y
            yum install mysql-shell -y
            setenforce 0
            systemctl start docker
            systemctl enable docker
            docker-compose -f /vagrant/docker-compose.yml up -d
          SHELL
          #box.vm.provision "ansible" do |ansible|
          #    ansible.compatibility_mode = "2.0"
          #    ansible.playbook = "replication.yml"
          #    ansible.verbose = "true"
          #    ansible.become = "true"
          #    ansible.limit = "all"
          #end
      end
  end
end
