Vagrant.configure("2") do |config|
  # Base box
  config.vm.box = "generic/ubuntu2204"
  config.vm.box_version = "4.3.12"

  config.vm.define "k8s-lab" do |node|
    node.vm.hostname = "k8s-lab"

    node.vm.network "private_network", ip: "192.168.56.10"

    node.vm.provider "virtualbox" do |vb|
      vb.name = "k8s-lab"
      vb.memory = 8192
      vb.cpus = 4
      vb.customize ["modifyvm", :id, "--vram", "64"]

      # Create a 64 GB virtual disk (added as secondary storage)
      vb.customize ["createhd", "--filename", "k8s-lab-disk.vdi", "--size", 65536]
      vb.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 1, "--device", 0, "--type", "hdd", "--medium", "k8s-lab-disk.vdi"]
    end

    config.ssh.forward_agent = true

    config.vm.synced_folder '.', '/vagrant', disabled: true

    config.vm.provision "docker" do |d|
      d.pull_images "jdxcode/mise"
    end

    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible/playbook.yml"
    end

    # [TODO]: #1 Move provisioner scripts to scripts directory

    wsl2_ssh_key_path = File.join(Dir.home, ".ssh", "id_rsa")

    if File.exist?(wsl2_ssh_key_path)
        wsl2_ssh_key = File.read(wsl2_ssh_key_path)
        
        config.vm.provision :shell, :inline => <<-SHELL
            echo 'WSL2-specific: Copying host primary SSH Key to VM for provisioning...'
            
            mkdir -p /root/.ssh
            echo '#{wsl2_ssh_key}' > /root/.ssh/id_rsa
            chmod 600 /root/.ssh/id_rsa
            chown root:root /root/.ssh/id_rsa
            
            mkdir -p /home/vagrant/.ssh
            echo '#{wsl2_ssh_key}' > /home/vagrant/.ssh/id_rsa
            chmod 600 /home/vagrant/.ssh/id_rsa
            chown vagrant:vagrant /home/vagrant/.ssh/id_rsa
            
            sudo su - vagrant -c 'ssh-keygen -R github.com'
            ssh-keyscan github.com >> /home/vagrant/.ssh/known_hosts
        SHELL
    else
        raise Vagrant::Errors::VagrantError, "\n\nERROR: Primary SSH Key not found at #{wsl2_ssh_key_path} (required on WSL2 host).\nEnsure your primary key is in this location.\n\n"
    end

    node.vm.provision "shell", inline: <<-SHELL
      echo "alias lab='cd /home/vagrant/lab'" >> /home/vagrant/.bash_aliases
      echo "alias k='kubectl'" >> /home/vagrant/.bash_aliases
      echo "alias pods='kubectl get pods'" >> /home/vagrant/.bash_aliases
      echo "alias deploy='kubectl apply -f'" >> /home/vagrant/.bash_aliases
    SHELL

    node.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      test -d ~/.linuxbrew && eval "$(~/.linuxbrew/bin/brew shellenv)"
      test -d /home/linuxbrew/.linuxbrew && eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
      echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/vagrant/.bashrc
      brew --version

      sudo -H -u vagrant git clone git@github.com:aadil96/lab.git /home/vagrant/lab
    SHELL
  end
end