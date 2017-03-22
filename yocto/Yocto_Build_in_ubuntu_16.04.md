### some workarounds for yocto building in ubuntu 16.04

Downgrade to gcc 4.8

	sudo apt-get install gcc-4.8 g++-4.8
	sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 100 --slave /usr/bin/g++ g++ /usr/bin/g++-4.8
	sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 50 --slave /usr/bin/g++ g++ /usr/bin/g++-5
	sudo update-alternatives --config gcc

Disable SDL support.

	sed -i 's/^PACKAGECONFIG_append_pn-qemu-native/#PACKAGECONFIG_append_pn-qemu-native/' conf/local.conf 
	sed -i 's/^PACKAGECONFIG_append_pn-nativesdk-qemu/#PACKAGECONFIG_append_pn-nativesdk-qemu/' conf/local.conf 

Enlarge pid number

	sysctl kernel.pid_max
	sysctl -w kernel.pid_max=4194303
