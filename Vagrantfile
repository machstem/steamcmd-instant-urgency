# -*- mode: ruby -*-
# vi: set ft=ruby :

## generic/debian9 image must already exist; e.g. d:\vagrant\vagrant.exe init generic/debian9
## Configure your virtual machine, using the generic/debian9 image

Vagrant.configure("2") do |config|
	config.vm.box = "generic/debian9"
## config.vm.provider can be used to tell vagrant which hypervisor product to access
	config.vm.provider "hyperv"
	config.vm.hostname = "insurgentserver"
## config.vm.network bridge: allows you to access your hyperv virtual switch, DHCP, network e.g. vSwitch
	config.vm.network "public_network", bridge: "vSwitch" 
	
	config.vm.provider "hyperv" do |hyperv|
		hyperv.vmname = "debian9-insurgency"
		hyperv.memory = 1500
		hyperv.maxmemory = 1500
		hyperv.cpus = 4
## Allows HyperV to start this virtual machine on host startup
		hyperv.auto_start_action = "Start"
		hyperv.mac ="00155d080208"
## Optional; uses hyperv's disk differencing technology
		hyperv.linked_clone = true
	end
## Use Linux Intergration Services (LIS) through the hyperv daemon
	config.vm.provider "hyperv" do |hyperv|
		hyperv.vm_integration_services = {
			guest_service_interface: true,
			shutdown: true,
			heartbeat: true
		}
	end
## Create new system level steam user, give him sudo access
## Download and extract the most recent steamcmd_linux client
	config.vm.provision "shell", inline: <<-SHELL
		useradd -rm -s /bin/bash steam
		usermod -aG sudo steam
		service ssh reload
		## Feel free to edit the password from 'steam' to something else.
		## Consider removing/disabling the local vagrant account
		echo -e "steam\nsteam" | passwd steam
		mkdir -p /home/steam/steamcmd/insurgency/Insurgency/Saved/Config/LinuxServer/
		mkdir -p /home/steam/tmp
		curl -o /home/steam/steamcmd/steamcmd_linux.tar.gz -sqL "https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz"
		tar -zxvf /home/steam/steamcmd/steamcmd_linux.tar.gz -C /home/steam/steamcmd/
		chown -R steam:steam /home/steam
		## Give all permissions for vagrant user to copy files into, during file provisioning
		chmod 0777 /home/steam/tmp
	SHELL

##  Copy files from <.\provisioning_files\steamcmd_insurgency> path
##
##  steamcmd launch and update scripts, and systemd service configuration file

	config.vm.provision "file",
		source: "D:/vagrant_builds/provisioning_files/steamcmd_insurgency/steamcmd_581330.service",
		destination: "/home/steam/tmp/steamcmd_581330.service"		
		
	config.vm.provision "file",
		source: "D:/vagrant_builds/provisioning_files/steamcmd_insurgency/insurgency_update",
		destination: "/home/steam/tmp/insurgency_update"
	
	config.vm.provision "file",
		source: "D:/vagrant_builds/provisioning_files/steamcmd_insurgency/insurgency_launch",
		destination: "/home/steam/tmp/insurgency_launch"
	
##  Insurgency Server configuration files
			
	config.vm.provision "file",
		source: "D:/vagrant_builds/provisioning_files/steamcmd_insurgency/server_configuration/Admins.txt",
		destination: "/home/steam/tmp/Admins.txt"
		
	config.vm.provision "file",
		source: "D:/vagrant_builds/provisioning_files/steamcmd_insurgency/server_configuration/Game.ini",
		destination: "/home/steam/tmp/Game.ini"

	config.vm.provision "file",
		source: "D:/vagrant_builds/provisioning_files/steamcmd_insurgency/server_configuration/MapCycle.txt",
		destination: "/home/steam/tmp/MapCycle.txt"

##  move files into respective directories

	config.vm.provision "shell", inline: <<-SHELL
		chown -R steam:steam /home/steam/tmp
		chmod +x /home/steam/tmp/insurgency_launch
		mv /home/steam/tmp/insurgency_update /home/steam/steamcmd/insurgency_update
		mv /home/steam/tmp/insurgency_launch /home/steam/steamcmd/insurgency_launch
		mv /home/steam/tmp/steamcmd_581330.service /home/steam/steamcmd/steamcmd_581330.service		
	SHELL

## adjust the following configurations to match your network's

	config.vm.provision "shell", inline: <<-SHELL
		echo "source /etc/network/interfaces.d/*" > /etc/network/interfaces
		echo "allow-hotplug eth0" >> /etc/network/interfaces
		echo "auto lo" >> /etc/network/interfaces
		echo "iface lo inet loopback" >> /etc/network/interfaces
		echo "iface eth0 inet dhcp" >> /etc/network/interfaces
		echo "dns-nameserver 192.168.1.1" >> /etc/network/interfaces
		echo "pre-up sleep 2"  >> /etc/network/interfaces
		echo "nameserver 192.168.1.1" > /etc/resolv.conf
		echo "search domain.com" >> /etc/resolv.conf
		echo "" >> /etc/initramfs-tools/modules
		## Enable hyperv modules for LIS
		echo "# Hyper-V Modules" >> /etc/initramfs-tools/modules
		echo "hv_vmbus" >> /etc/initramfs-tools/modules
		echo "hv_storvsc" >> /etc/initramfs-tools/modules
		echo "hv_blkvsc" >> /etc/initramfs-tools/modules
		echo "hv_netvsc" >> /etc/initramfs-tools/modules
		echo "hv_balloon" >> /etc/initramfs-tools/modules
		echo "hv_utils" >> /etc/initramfs-tools/modules
		update-initramfs -ut
		echo "" >> /etc/apt/sources.list
		echo "deb http://deb.debian.org/debian stretch main contrib non-free" >> /etc/apt/sources.list
		echo "deb-src http://deb.debian.org/debian stretch main contrib non-free" >> /etc/apt/sources.list
		echo "" >> /etc/apt/sources.list
		echo "deb http://deb.debian.org/debian stretch-updates main contrib non-free" >> /etc/apt/sources.list
		echo "deb-src http://deb.debian.org/debian stretch-updates main contrib non-free" >> /etc/apt/sources.list
		echo "" >> /etc/apt/sources.list
		echo "deb http://security.debian.org/ stretch/updates main contrib non-free" >> /etc/apt/sources.list
		echo "deb-src http://security.debian.org/ stretch/updates main contrib non-free" >> /etc/apt/sources.list
		dpkg --add-architecture i386		
		apt-get update
		##  install basic troubleshooting tools and required libraries for steamcmd
		apt-get install -y net-tools vim tcpdump lib32gcc1 htop iftop
		##  install Insurgency Sandstorm from the file at /home/steam/steamcmd/insurgency_update
		runuser -l steam -c '/home/steam/steamcmd/steamcmd.sh +runscript /home/steam/steamcmd/insurgency_update'
	SHELL
##  move all the appropriate game server files into their correct location
##  create service for update/launch
##  adjust to run on startup
	config.vm.provision "shell", inline: <<-SHELL
		mv -f /home/steam/tmp/Admins.txt /home/steam/steamcmd/insurgency/Insurgency/Saved/Config/LinuxServer/Admins.txt
		mv -f /home/steam/tmp/Game.ini /home/steam/steamcmd/insurgency/Insurgency/Saved/Config/LinuxServer/Game.ini
		mv -f /home/steam/tmp/MapCycle.txt /home/steam/steamcmd/insurgency/Insurgency/Saved/Config/LinuxServer/MapCycle.txt
		ln -s /home/steam/steamcmd/steamcmd_581330.service /etc/systemd/system/steamcmd_581330.service
		systemctl daemon-reload
		systemctl enable steamcmd_581330.service
		systemctl start steamcmd_581330.service
	SHELL
	
	:reload
	config.vm.post_up_message 
end
