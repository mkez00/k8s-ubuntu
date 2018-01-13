# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
	config.vm.define "master" do |c|
		c.vm.hostname = 'master'
		c.vm.box = "ubuntu/xenial64"
		c.vm.synced_folder "data", "/data"
		c.vm.network "private_network", ip: "192.168.56.10"
		c.vm.provision "shell", inline: <<-SHELL
			sudo su

			# Need to ensure that hostname -i returns a routable IP address.  See: https://kubernetes.io/docs/setup/independent/troubleshooting-kubeadm/
			sed 's/127\.0\.0\.1.*master.*/192\.168\.56\.10 master/' -i /etc/hosts

			apt-get update 
			apt-get install -y apt-transport-https
			curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
			cat <<EOF > /etc/apt/sources.list.d/kubernetes.list  
deb http://apt.kubernetes.io/ kubernetes-xenial main  
EOF
			apt-get update
			apt-get install -y docker.io
			apt-get install -y kubelet kubeadm kubectl kubernetes-cni

			# need to avertise API server address on interface that is reachable for other nodes.  Storing join command from output in shell script for use by worker nodes
			kubeadm init --apiserver-advertise-address=192.168.56.10 --token=b9e6bb.6746bcc9f8ef8267 --pod-network-cidr=10.244.0.0/16  | grep "kubeadm join --token" > /data/join-cluster.sh

			# setup kubectl for root
      		mkdir -p /root/.kube
      		cp -i /etc/kubernetes/admin.conf /root/.kube/config

      		# setup kubectl for ubuntu user
      		mkdir -p /home/ubuntu/.kube
      		cp -i /etc/kubernetes/admin.conf /home/ubuntu/.kube/config
      		chown ubuntu:ubuntu /home/ubuntu/.kube/config

      		sysctl net.bridge.bridge-nf-call-iptables=1

      		# using slightly modified flannel config since it requires using enp0s8 as default interface.  See: https://github.com/kubernetes/kubeadm/issues/139#issuecomment-276607463
      		kubectl apply -f /data/kube-flannel.yml

      		# be sure that kube-dns pod is up and running before adding nodes
      		# kubectl get pods --all-namespaces
      		kubectl run spring-boot-hello --image=docker.io/mkez00/spring-boot-docker:latest --port=8080

      		# Expose deployment with NodePort type, see types and descriptions at: https://kubernetes.io/docs/tutorials/kubernetes-basics/expose-intro/
      		#kubectl expose deployment spring-boot-hello --name=spring-boot-hello

		SHELL
	end

	$node_script = <<-SHELL
		sudo su

		# Need to ensure that hostname -i returns a routable IP address.  See: https://kubernetes.io/docs/setup/independent/troubleshooting-kubeadm/
		sed "s/127\.0\.0\.1.*node${1}.*/192\.168\.56\.${1} node${1}/" -i /etc/hosts

		apt-get update 
		apt-get install -y apt-transport-https
		curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
		cat <<EOF > /etc/apt/sources.list.d/kubernetes.list  
deb http://apt.kubernetes.io/ kubernetes-xenial main  
EOF
		apt-get update
		apt-get install -y docker.io
		apt-get install -y kubelet kubeadm kubectl kubernetes-cni

		sysctl net.bridge.bridge-nf-call-iptables=1

		# Join node to cluster
		sh /data/join-cluster.sh
	SHELL

	$num_node = 2
	$num_node.times do |n|
		node_vm_name = "node#{n+20}"
		hostname = "node#{n+20}"
		private_ip = "192.168.56.#{n+20}"
		node = "#{n+20}"

		config.vm.define node_vm_name do |c|
			c.vm.hostname = hostname
			c.vm.box = "ubuntu/xenial64"
			c.vm.synced_folder "data", "/data"
			c.vm.network "private_network", ip: private_ip
			c.vm.provision "shell", run: "always" do |s|
				s.inline = $node_script
				s.args = node
			end
		end
	end

end