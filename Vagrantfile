# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.provider :virtualbox do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end

  config.vm.synced_folder ".", "/vagrant", type: "virtualbox"

  config.vm.provision "shell", inline: <<-SHELL
    export DEBIAN_FRONTEND=noninteractive
    # Update and upgrade the system
    sudo add-apt-repository ppa:gns3/ppa
    sudo apt-get update && sudo apt-get upgrade -y

    # Install dependencies
    sudo DEBIAN_FRONTEND=noninteractive apt-get install -y gns3-server docker.io

    sudo usermod -aG ubridge,libvirt,kvm,docker $(whoami)

    # Start Docker service
    echo "🐳 Starting Docker service..."
    sudo systemctl start docker

    # Enable Docker service
    echo "🐳 Enabling Docker service..."
    sudo systemctl enable docker

    # Verify Docker installation
    docker --version

    # Run GNS3
    echo "🚀 Running GNS3..."
    gns3server &

    # Build Docker images
    cd /vagrant
    echo "🚀 Building Docker images..."
    docker build -t p1-alpine -f P1/Dockerfile.alpine P1/
    docker build -t p1-frr -f P1/Dockerfile.frr P1/

    # P1 -- A router and a host
    curl -X POST http://localhost:3080/v2/templates -H "Content-Type: application/json" -d @/vagrant/P1/router_template.json
    curl -X POST http://localhost:3080/v2/templates -H "Content-Type: application/json" -d @/vagrant/P1/host_template.json

    sleep 1 # Import the project
    curl -X POST http://localhost:3080/v2/projects/12345678-123e-12c3-1c23-000000000000/import?name=p1 -H "Content-Type: multipart/form-data" -F "file=@/vagrant/P1/p1.gns3project"
    sleep 1 # Start the project
    curl -X POST http://localhost:3080/v2/projects/12345678-123e-12c3-1c23-000000000000/open
    sleep 1 # Start all nodes
    curl -X POST http://localhost:3080/v2/projects/12345678-123e-12c3-1c23-000000000000/nodes/start


    echo -e "\e[34m████████████████████████████████████████████████████████████\e[0m"
    # P2 -- VXLAN static and dynamic
    sleep 1 # Import the project
    curl -X POST http://localhost:3080/v2/projects/12345678-123a-12a3-1a23-000000000000/import?name=p2 -H "Content-Type: multipart/form-data" -F "file=@/vagrant/P2/p2.gns3project"

    echo -e "\e[34m████████████████████████████████████████████████████████████\e[0m"
    # P3 -- BGP-EVPN Spine-and-Leaf Architecture
    sleep 1 # Import the project
    curl -X POST http://localhost:3080/v2/projects/12345678-123b-12b3-1b23-000000000000/import?name=p3 -H "Content-Type: multipart/form-data" -F "file=@/vagrant/P3/p3.gns3project"
    
  SHELL

  config.vm.define "bgp" do |server|
    server.vm.hostname = "bgp"
    server.vm.network "forwarded_port", guest: 3080, host: 3081
  end
end
