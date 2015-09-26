# coreos-Azure

##Setup

####Create SSH public/private key pair
[Instructions](https://winscp.net/eng/docs/ui_puttygen)

[Information](http://the.earth.li/~sgtatham/putty/0.60/htmldoc/Chapter8.html)

####Install Azure xplat cli
[Instructions](https://azure.microsoft.com/en-us/documentation/articles/xplat-cli/)

####Clone this repo
	git clone https://github.com/ssugar/coreos-Azure

####Modify the cloud-config.yaml

Modify the following fields:
 - hostname: change X to the node number
 - ssh_authorized_keys: you can use normal RSA SSH public key format
 - discovery: change \<token\> to the token you generate at https://discovery.etcd.io/new?size=X where X = the desired size of your new cluster

		#cloud-config

		hostname: node-X

		ssh_authorized_keys:
		  - ssh-rsa AAAAA......

		coreos:
		  etcd2:
			# generate a new token for each unique cluster from https://discovery.etcd.io/new?size=X
			# specify the initial size of your cluster with ?size=X
			discovery: https://discovery.etcd.io/<token>
			advertise-client-urls: http://$public_ipv4:2379
			initial-advertise-peer-urls: http://$public_ipv4:2380
			listen-client-urls: http://0.0.0.0:2379
			listen-peer-urls: http://0.0.0.0:2380
		  units:
			- name: etcd2.service
			  command: start
			- name: fleet.service
			  command: start
		  
		write_files:
		  - path: /etc/ssh/sshd_config
			permissions: 0600
			owner: root:root
			content: |
			  # Use most defaults for sshd configuration.
			  UsePrivilegeSeparation sandbox
			  Subsystem sftp internal-sftp
			  PermitRootLogin no
			  AllowUsers core
			  PasswordAuthentication no
			  ChallengeResponseAuthentication no

####Create the cloud service

    azure service create coreosA
	
	azure service create coreosB

####Bring up the VMs

First Node:

    azure vm create --custom-data=cloud-config.yaml --ssh=22 --vm-name=coreosA --connect=coreosA 2b171e93f07c4903bcad35bda10acf22__CoreOS-Stable-766.3.0 core

Second Node:

    azure vm create --custom-data=cloud-config.yaml --ssh=22 --vm-name=coreosB --connect=coreosB 2b171e93f07c4903bcad35bda10acf22__CoreOS-Stable-766.3.0 core

####Open up the VM endpoints

    azure vm endpoint create coreosA 2379 2379
    azure vm endpoint create coreosA 2380 2380

    azure vm endpoint create coreosB 2379 2379
    azure vm endpoint create coreosB 2380 2380