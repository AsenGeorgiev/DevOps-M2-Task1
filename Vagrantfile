# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
    
  config.ssh.insert_key = false

  config.vm.define "docker" do |docker|
    docker.vm.box="shekeriev/debian-11"
    docker.vm.hostname = "docker"
    docker.vm.network "private_network", ip: "192.168.99.100"
    docker.vm.network "forwarded_port", guest: 80, host: 8000, auto_correct: true
    docker.vm.network "forwarded_port", guest: 8080, host: 8080, auto_correct: true
    docker.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "2048"]
    end

    docker.vm.provision "shell", inline: <<EOS
echo "* Add hosts ..."
echo "192.168.99.100 docker" >> /etc/hosts

echo "* Add any prerequisites ..."
apt-get update
apt-get install -y ca-certificates curl gnupg lsb-release

echo "* Add Docker repository and key ..."
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

echo "* Install Docker ..."
apt-get update
apt-get install -y docker-ce docker-ce-cli containerd.io
 
echo "* Add vagrant user to docker group ..."
usermod -aG docker vagrant

echo "* Adjust Docker configuration ..."
sed -i 's@-H fd://@-H fd:// -H tcp://0.0.0.0@g' /lib/systemd/system/docker.service

echo "* Restart Docker ..."
systemctl daemon-reload
systemctl restart docker
EOS
  end

  config.vm.define "ans" do |ans|
    ans.vm.box = "shekeriev/centos-stream-9"
    ans.vm.hostname = "ans.do2.lab"
    ans.vm.network "private_network", ip: "192.168.99.99"
    ans.vm.synced_folder "vagrant/", "/vagrant"
    ans.vm.provision "shell", inline: <<EOS
dnf install -y ansible-core git
ansible-galaxy collection install -p /usr/share/ansible/collections ansible.posix

 echo "Adding webserver to hosts..."
 sudo sed -i '$ a\ 192.168.99.100 web' /etc/hosts

 echo "Clone gitfiles..."
 cd /etc/ansible/
 git clone https://github.com/AsenGeorgiev/Task1.git
 cd /etc/ansible/Task1
 sudo cp hosts /etc/ansible/

 echo "Files deployment..."
 cd /etc/ansible/Task1
 sudo cp ansible.cfg inventory playbook.yml /home
 cd /home

 echo "Execute playbook..."
 cd /home
 ansible-playbook playbook.yml
EOS
  end
end