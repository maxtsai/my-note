### Quick setting to enable samba
<pre>
	/etc/samba/smb.conf

	[global]
		workgroup = WORKGROUP
		server string = Samba Server %v
		netbios name = max-workstation
		security = share
		map to guest = bad user
		name resolve order = bcast host
		dns proxy = no

	[public]
		path = /home/max/samba/public
		public = yes
		writable = yes
		guest account = max
		force user = max
		force group = max
		browsable = yes

	[public_tmp]
		path = /home/max/samba/tmp
		public = yes
		writable = yes
		guest account = max
		force user = max
		force group = max
		browsable = yes
</pre>
