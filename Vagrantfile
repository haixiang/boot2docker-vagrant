Vagrant.require_version ">= 1.6.3"

Vagrant.configure("2") do |config|
  config.vm.define "boot2docker"

  config.vm.box = "dduportal/boot2docker"
  config.vm.box_check_update = false

  # The default box private network IP is 192.168.10.10
  # Uncomment lines below to map additional IP addresses for use with multiple projects.
  # Project specific IP:port mapping for containers is done in via docker-compose (docker-compose.yml)
  #config.vm.network "private_network", ip: "192.168.10.11"
  #config.vm.network "private_network", ip: "192.168.10.12"
  #config.vm.network "private_network", ip: "192.168.10.13"

  # Synced folder setup
  if Vagrant::Util::Platform.windows?
    # Default/Windows mount (vboxfs)
    config.vm.synced_folder ".", "/vagrant"
    # Uncomment for better performance on Windows (mount via SMB).
    # Requires Vagrant to be run with admin privileges.
    # Will also prompt for the Windows username and password to access the share.
    #config.vm.synced_folder ".", "/vagrant", type: "smb"
  else
    # Mount Vagrantfile directory (<Project_XYZ> or a shared <Projects> folder) under the same path in the VM.
    # Required for the docker-compose client to work from the OSX host.
    # NFS mount works much-much faster on OSX compared to the default vboxfs.
    # See https://github.com/mitchellh/vagrant/issues/2304 for why NFS over TCP may be better than over UDP.
    vagrant_root = File.dirname(__FILE__)
    #config.vm.synced_folder vagrant_root, vagrant_root, type: "nfs", mount_options: ["nolock", "vers=3", "udp"]
    config.vm.synced_folder vagrant_root, vagrant_root, type: "nfs", mount_options: ["nolock", "vers=3", "tcp"]
    
    # This uses uid and gid of the user that started vagrant.
    config.nfs.map_uid = Process.uid
    config.nfs.map_gid = Process.gid
  end
  
  # CPU and memory settings
  config.vm.provider "virtualbox" do |v|
    v.cpus = 1  # VirtualBox works much better with a single CPU.
    v.memory = 2048
  end

  # Allow Mac OS X docker client to connect to Docker without TLS auth
  # https://github.com/deis/deis/issues/2230#issuecomment-72701992
  config.vm.provision "shell" do |s|
    s.inline = <<-SCRIPT
      echo 'DOCKER_TLS=no' >> /var/lib/boot2docker/profile
      sudo /etc/init.d/docker stop >/dev/null 2>&1 || sudo /etc/init.d/docker start
    SCRIPT
  end

  # Make docker-compose available inside boot2docker via a container.
  # https://github.com/docker/compose/issues/598#issuecomment-67762456
  config.vm.provision "shell", run: "always" do |s|
    s.inline = <<-SCRIPT
      echo 'alias docker-compose='"'"'docker run --rm -it \
          -v $(pwd):$(pwd) -v /var/run/docker.sock:/var/run/docker.sock \
          -e COMPOSE_PROJECT_NAME=$(basename $(pwd)) -w="$(pwd)" \
          blinkreaction/docker-compose'"'" >> /home/docker/.ashrc
      echo 'alias fig=docker-compose' >> /home/docker/.ashrc
    SCRIPT
  end

end
