### some workarounds for yocto building in ubuntu 16.04

Ubuntu uses gcc-5 by default but some yocto (branch imx-3.14.28-1.0.1_patch) packages aren't compatible with this compiler so we need to switch to older one.

	sudo apt-get install gcc-4.8 g++-4.8
	sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 100 --slave /usr/bin/g++ g++ /usr/bin/g++-4.8
	sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 50 --slave /usr/bin/g++ g++ /usr/bin/g++-5
	sudo update-alternatives --config gcc

Ubuntu uses newer version of the SDL libraries which are not compatible with yocto (branch imx-3.14.28-1.0.1_patch).
This commands needs to be run after every "source \*" command to commend out SDL support.

	sed -i 's/^PACKAGECONFIG_append_pn-qemu-native/#PACKAGECONFIG_append_pn-qemu-native/' conf/local.conf 
	sed -i 's/^PACKAGECONFIG_append_pn-nativesdk-qemu/#PACKAGECONFIG_append_pn-nativesdk-qemu/' conf/local.conf 

Extend pid number

	sysctl kernel.pid_max
	sysctl -w kernel.pid_max=4194303
