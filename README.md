# Overview

Demonstrates provisioning a Kubernetes cluster with a single master and any number of worker nodes (default 2) using Ubuntu as Linux distro and VirtualBox as hypervisor.  This Kubernetes cluster uses Flannel for its CNI implementation.  

# Requirements

1) VirtualBox or other type 2 hypervisor
2) Vagrant (by Hashicorp)

# Instructions

1) Run `vagrant up` from root of project (where Vagrantfile is located)
2) After completed, SSH into master using `vagrant ssh master` and into worker nodes with `vagrant ssh node#` where # is the node number (1,2,3...n)

# Verify Pod Communication

1) SSH into master node using `vagrant ssh master`
2) Verify all pods are up and running with `kubectl get pods --all-namespaces`.  All should have a STATUS of Running.
3) For the sample "Hello World" deployment that has been provisioned.  Get the IP address of the pod by running `kubectl describe pods spring-boot-hello-xxxxxxxxxx` (copy the pod name from the pod list in step 2)
4) Copy the IP listed in the output of the describe command
5) Run `kubectl run curl --image=radial/busyboxplus:curl -i --tty`
6) Run `curl <COPIED_POD_IP>:8080`.  'Hello Docker World' is the returned output

# Exposing Deployment with Services

1) SSH into the master node using `vagrant ssh master`
2) Run `kubectl expose deployment spring-boot-hello --name=spring-boot-hello`
3) Run `kubectl get svc`.  Copy the CLUSTER-IP and PORT for the `spring-boot-hello` service
4) Run `curl <CLUSTER_IP>:<PORT>`.  'Hello Docker World' is the returned output
5) SSH into any of the worker nodes with `vagrant ssh node#` where # is the node number (1,2,3...n)
6) Run `curl <CLUSTER_IP>:<PORT>`.  'Hello Docker World' is the returned output

# Utilizing Kube DNS for Pod-to-Pod Communication

Pods often need to communicate.  For example, if you're running Prometheus for monitoring and Grafana as your visualization utility.  Grafana will need a connection to Prometheus.  Using DNS is ideal vs tracking pod or exposed service IP addresses.

1) Complete steps 1 and 2 from 'Exposing Deployment with Services' section
2) While still in the master node run `kubectl run curl --image=radial/busyboxplus:curl -i --tty`.  If this errors with an AlreadyExists error.  Get the Pod name (`kubectl get pods` and run `kubectl attach <POD_NAME> -c curl -i -t`)
3) Run `curl spring-boot-hello:8080`.  'Hello Docker World' is the returned output

