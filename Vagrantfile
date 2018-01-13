# -*- mode: ruby -*-
# vi: set ft=ruby :

#If you are using VirtualBox (directly or via Vagrant), you will need to ensure that hostname -i returns a routable IP address (i.e. one on the second network interface, not the first one). By default, it doesn’t do this and kubelet ends-up using first non-loopback network interface, which is usually NATed. Workaround: Modify /etc/hosts, take a look at this Vagrantfileubuntu-vagrantfile for how this can be achieved

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
			# verify hostname
			# hostname -i

			apt-get update 
			apt-get install -y apt-transport-https
			curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
			cat <<EOF > /etc/apt/sources.list.d/kubernetes.list  
deb http://apt.kubernetes.io/ kubernetes-xenial main  
EOF
			apt-get update
			apt-get install -y docker.io
			apt-get install -y kubelet kubeadm kubectl kubernetes-cni

			kubeadm init --apiserver-advertise-address=192.168.56.10 --token=b9e6bb.6746bcc9f8ef8267 --pod-network-cidr=10.244.0.0/16

			# setup kubectl for root
      		mkdir -p /root/.kube
      		cp -i /etc/kubernetes/admin.conf /root/.kube/config

      		# setup kubectl for ubuntu user
      		mkdir -p /home/ubuntu/.kube
      		cp -i /etc/kubernetes/admin.conf /home/ubuntu/.kube/config
      		chown ubuntu:ubuntu /home/ubuntu/.kube/config

      		# kubectl get nodes
      		# At this point node is listed as NotReady
      		# kubectl describe nodes master
      		# Under Conditions > Ready.  Value will be False, states that CNI not configured: runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized

      		sysctl net.bridge.bridge-nf-call-iptables=1

      		# using slightly modified flannel config since it requires using enp0s8 as default interface.  See: https://github.com/kubernetes/kubeadm/issues/139#issuecomment-276607463
      		kubectl apply -f /data/kube-flannel.yml


      		# be sure that kube-dns pod is up and running before adding nodes
      		# kubectl get pods --all-namespaces

      		# kubectl get nodes
      		# Node will be listed as Ready

      		#kubectl run spring-boot-hello --image=docker.io/mkez00/spring-boot-docker:latest --port=8080

      		# Expose deployment with NodePort type, see types and descriptions at: https://kubernetes.io/docs/tutorials/kubernetes-basics/expose-intro/
      		#kubectl expose deployment spring-boot-hello --name=spring-boot-hello

		SHELL
	end

	config.vm.define "node1" do |c|
		c.vm.hostname = 'node1'
		c.vm.box = "ubuntu/xenial64"
		c.vm.network "private_network", ip: "192.168.56.11"
		c.vm.provision "shell", inline: <<-SHELL
			sudo su

			# Need to ensure that hostname -i returns a routable IP address.  See: https://kubernetes.io/docs/setup/independent/troubleshooting-kubeadm/
			sed 's/127\.0\.0\.1.*node1.*/192\.168\.56\.11 node1/' -i /etc/hosts

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
			#kubeadm join --token b9e6bb.6746bcc9f8ef8267 192.168.56.10:6443 --discovery-token-ca-cert-hash sha256:f65387667c28389994b5642c2f5ffa6d29068c05cd6a506cc853591c7e348424
		SHELL
	end

	config.vm.define "node2" do |c|
		c.vm.hostname = 'node2'
		c.vm.box = "ubuntu/xenial64"
		c.vm.network "private_network", ip: "192.168.56.12"
		c.vm.provision "shell", inline: <<-SHELL
			sudo su

			# Need to ensure that hostname -i returns a routable IP address.  See: https://kubernetes.io/docs/setup/independent/troubleshooting-kubeadm/
			sed 's/127\.0\.0\.1.*node2.*/192\.168\.56\.12 node2/' -i /etc/hosts

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
			#kubeadm join --token b9e6bb.6746bcc9f8ef8267 192.168.56.10:6443 --discovery-token-ca-cert-hash sha256:f65387667c28389994b5642c2f5ffa6d29068c05cd6a506cc853591c7e348424
		SHELL
	end
end