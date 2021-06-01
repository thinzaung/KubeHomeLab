# Kubnertes Clusters in Home Lab
Following is the instruction on how kubernetes clusters are created in my home network. 

- Kubernetes Version: 1.19.1 and upgraded to 1.20.7
- Docker Version: 20.10.2
- Plugins Used: Calico v3.19.1

<i>Note: 
* I run the following VMs on based Redhat Baremetal box with 1 physcial core ad 36GB memory. 
* For DNS resolution, I use Local Hosts file </i>

## Part I -  Creating Kubernetes clusters using Vagrant Boxes: 

1. Create Vagrant Boxes using following Vagrant File : 

* OS: ubuntu Focal 64 bit
* VM provider : Virtual Box
* CPU/Memory : 2 core / 8GB
* Network : Brige Adatpaer in Virtual box
    * While configuring Compute with Vagrant file, please make sure the box is configured only with Bridge Network Adapater 
    * If multiple adapaters are creted, Kubernetes will pick up the first network adapter which might not be able to 

Example Vagrant File below. Please replace hostname, network adapater and usernames repsectively</br>

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.
  config.vm.hostname="sbdev2001"
  config.ssh.host="sbdev2001"
  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/focal64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "public_network", :adapter=>1, bridge: "en01"
  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
     vb.customize ["modifyvm", :id, "--memory", "5140", "--cpus", "2"]
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
  config.vm.provision "shell", inline: <<-SHELL
     apt-get update
     apt install net-tools
     useradd -m -d /home/thinny81 thinny81
     usermod -aG root thinny81
     usermod -aG sudo thinny81
     newuser=`grep thinny81 /etc/passwd`
     echo "added $newuser"
     cp /etc/ssh/sshd_config /etc/ssh/sshd_config_orig
     sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
  SHELL
end
```

## Part II -  Setting up Kubernetes Cluster  ( 1 master and 2 worker nodes)
### 1. Setting up master node 
Please run the following commands as <code>root</code>
- Update and upgrade kernel : 
```
apt-get update && apt-get upgrade -y
```
- Install docker
```
apt-get install -y docker.io
```
- Add Kubernetes Repo and Add GPG key for the packages
```
thinny81@kubemaster2001:~$ cat /etc/apt/sources.list.d/kubernetes.list
deb  http://apt.kubernetes.io/  kubernetes-xenial  main

curl -s \
   https://packages.cloud.google.com/apt/doc/apt-key.gpg \
   | apt-key add - 

apt-get update
```
- Install kubeadm, kubelete and kubectl
```
apt-get install -y \
        kubeadm=1.19.1-00 kubelet=1.19.1-00 kubectl=1.19.1-00
```
- Mark hold kubeadm, kubelet and kubectl from accidentally upgrading or applying updates
```
apt-mark hold kubelet kubeadm kubectl
```
- Download Calico configuration file and check POD CIDR
```
wget https://docs.projectcalico.org/manifests/calico.yaml

```
- Kubeadm init
```
kubeadm init --apiserver-advertise-address=10.0.0.37 \ 
     --apiserver-cert-extra-sans=10.0.0.37 \ 
     --node-name kubemaster2001 \ 
     --pod-network-cidr=192.168.0.0/16 \ 
     --control-plane-endpoint="kubemaster2001:6443"
```
- Please take note of the folloing JOIN commands to join the cluster 
(if we do not remember these join commands, we can regenerate them too)
```
To join as second control plane node : 

kubeadm join kubemaster2001:6443 \ 
--token gzbaks.0e1bbbk00utk88u7 \ 
--discovery-token-ca-cert-hash \ sha256:6a4e09604ab314622d0e52774e245ede36aea3fc9b53ea593cc3b9be9fe5c556 \ 
--control-plane \ 
--certificate-key d76b6cd2b3a41495cddc436c1a52eb813041acd4fafa3294493720a584e682f4

To Join as a worker node: 

kubeadm join kubemaster2001:6443 \ 
--token 06leg5.9aqlax2j6184d0wf \ 
--discovery-token-ca-cert-hash sha256:6a4e09604ab314622d0e52774e245ede36aea3fc9b53ea593cc3b9be9fe5c556
```

- If tokens are expired or need to recreate: (Please run the following in master node,  you can run as <code>root</code> or <code> sudo [command] </code>)
```
kubeadm token list # <= to view the current token list

thinny81@kubemaster2001:~$ sudo kubeadm token create
4q5498.ripn7fu0bw0fasyf

thinny81@kubemaster2001:~$ openssl x509 -pubkey \ 
-in /etc/kubernetes/pki/ca.crt  | openssl rsa -pubin \ 
-outform der 2>/dev/null | openssl dgst -sha256 \ 
-hex

Output: 
----------
(stdin)= 6a4e09604ab314622d0e52774e245ede36aea3fc9b53ea593cc3b9be9fe5c556

root@kubeworker2002: ̃# kubeadm join \
     --token 4q5498.ripn7fu0bw0fasyf \
     kubemaster2001:6443 \
     --discovery-token-ca-cert-hash \
     sha256:6a4e09604ab314622d0e52774e245ede36aea3fc9b53ea593cc3b9be9fe5c556

Or you can create : 
========================================
[[ For joining as 2nd Control Plane]]:
========================================
thinny81@kubemaster2001:~$ sudo kubeadm token create \ 
--print-join-command \ 
--certificate-key \ 
$(kubeadm alpha certs certificate-key)

Output: 
---------
kubeadm join \ 
kubemaster2001:6443 \ 
--token 5c5f9u.t9rvgl3ddiamcygi \ 
--discovery-token-ca-cert-hash sha256:6a4e09604ab314622d0e52774e245ede36aea3fc9b53ea593cc3b9be9fe5c556 \ 
--control-plane \ 
--certificate-key 19729a5253cd0254c9928e3a6f018fa06c4255d4b1f27a3b56561abb98733907
========================================
[[For Joining as another node]]: 
========================================
thinny81@kubemaster2001:~$ sudo kubeadm token create\ 
--print-join-command

Output:
---------
kubeadm join kubemaster2001:6443 \ 
--token o15fkd.onqtk4hpla0lr5n0 \ 
--discovery-token-ca-cert-hash sha256:6a4e09604ab314622d0e52774e245ede36aea3fc9b53ea593cc3b9be9fe5c556

```
### 2. Setting up KubeConfig 
Setup the right kubernetes configuration file so that it will connect to the right cluser
```
Please run the following commands as non root user

mkdir -p ~$HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

You can set the kubeconfig as following: 
export KUBECONFIG=~/.kube/admin.conf

Or You can set the alias in your bash_profile so that if you have different clusters you can connect to the right one: 

alias k8dev="export KUBECONFIG=~/.kube/kube-test.conf"

```

### 3. Setting up network plugin configuraiton to the cluseter
```
download the calico configuration yaml file : 
wget https://docs.projectcalico.org/manifests/calico.yaml

kubectl apply -f calico.yaml

```

### 4. Growing Cluster - adding worker nodes
Please run the following commands as <code>root</code>
- Update and upgrade kernel : 
```
apt-get update && apt-get upgrade -y
```
- Install docker
```
apt-get install -y docker.io
```
- Add Kubernetes Repo and Add GPG key for the packages
```
thinny81@kubeworker2002:~$ cat /etc/apt/sources.list.d/kubernetes.list
deb  http://apt.kubernetes.io/  kubernetes-xenial  main

curl -s \
   https://packages.cloud.google.com/apt/doc/apt-key.gpg \
   | apt-key add - 

apt-get update
```
- Install kubeadm, kubelete and kubectl
```
apt-get install -y \
        kubeadm=1.19.1-00 kubelet=1.19.1-00 kubectl=1.19.1-00
```
- Mark hold kubeadm, kubelet and kubectl from accidentally upgrading or applying updates
```
apt-mark hold kubelet kubeadm kubectl
```
- Join the worker node in the cluster using the join command above : 
```
kubeadm join kubemaster2001:6443 \ 
--token o15fkd.onqtk4hpla0lr5n0 \ 
--discovery-token-ca-cert-hash sha256:6a4e09604ab314622d0e52774e245ede36aea3fc9b53ea593cc3b9be9fe5c556
```

- Verify the cluster  
```
kubectl get nodes
```

### 5. Run NGINX Pod in the cluster
```
kubectl create deployment nginx --image=nginx

kubectl get deployment nginx -o yaml > first.yaml
```
Add the followings in <code>first.yaml</code>
```
 spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        ports:                   # Add these
        - containerPort: 80      # three
          protocol: TCP          # lines
```
Replace the deployment and expose services so that we can access the service outside of the cluseters
```
kubectl replce -f first.yaml

kubectl expose deployment nginx --type=LoadBalancer

kubectl get svc
```