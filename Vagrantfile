$setup_docker = <<SCRIPT
apt-get update;
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -;
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt-get update;
apt-get install -y docker-ce docker-ce-cli containerd.io;
sed -i 's|-H fd://|-H fd:// -H tcp://0.0.0.0:2376|g' /lib/systemd/system/docker.service;
sudo usermod -aG docker vagrant
systemctl daemon-reload && systemctl restart docker.service;
SCRIPT

$swarm_init_script = <<SCRIPT
echo Swarm Init...
#sudo docker swarm init --listen-addr 10.1.5.10:2376 --advertise-addr 10.1.5.10:2376
docker -H tcp://10.1.5.10:2376 swarm init --advertise-addr 10.1.5.10
docker -H tcp://10.1.5.10:2376 swarm join-token -q worker > /vagrant/worker_token
docker -H tcp://10.1.5.11:2376 swarm join --token $(cat /vagrant/worker_token) 10.1.5.10:2376
docker -H tcp://10.1.5.12:2376 swarm join --token $(cat /vagrant/worker_token) 10.1.5.10:2376
#sudo docker swarm join-token --quiet worker > /vagrant/worker_token
SCRIPT

Vagrant.configure(2) do |config|
  (1..2).each do |i|
    config.vm.define "worker0#{i}" do |config|
      config.vm.box = "ubuntu/bionic64"
      config.vm.hostname = "worker0#{i}"
      config.vm.network "private_network", ip: "10.1.5.1#{i}"
      config.vm.provision "shell", inline: $setup_docker
    end
  end
  
  config.vm.define "manager" do |config|
    config.vm.box = "ubuntu/bionic64"
    config.vm.hostname = "manager"
    config.vm.network "private_network", ip: "10.1.5.10"
    config.vm.provision "shell", inline: $setup_docker
    config.vm.provision "shell", inline: $swarm_init_script, privileged: true
  end

end