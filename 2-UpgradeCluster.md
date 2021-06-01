## Part III  -  Upgrading Kubernetes Clusters

#### In this section, we will upgrade kubernetes to 1.20.7 from version 1.19.1. The Workflow will be : 
- Upgrade the control plane node
- Upgrade additional control plane nodes
- Upgrade worker nodes

Since the current cluster we setup is 1.19.1, we will need to upgrade the environment in 2 steps. 
- First we will need to upgrade the cluster to 1.19.11 which is the latest version in 1.19.x 
- Then we will upgrade the cluster to 1.20.x 

### 1. Upgrade Kubeadm on Master  Node
- First upgrade Kubeadm
<br>
> apt-mark unhold kubeadm && \
> apt-get update && apt-get install -y kubeadm=1.19.11-00 && \
> apt-mark hold kubeadm

```
Output
=======
kubeadm was already not hold.
Get:2 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Hit:3 http://archive.ubuntu.com/ubuntu focal InRelease
Hit:1 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Get:4 http://archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Get:5 http://archive.ubuntu.com/ubuntu focal-backports InRelease [101 kB]
Fetched 328 kB in 3s (116 kB/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 8 not upgraded.
Need to get 0 B/7758 kB of archives.
After this operation, 49.2 kB disk space will be freed.
(Reading database ... 130213 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.19.11-00_amd64.deb ...
Unpacking kubeadm (1.19.11-00) over (1.19.1-00) ...
Setting up kubeadm (1.19.11-00) ...
kubeadm set on hold.

Verify
=========
root@kubemaster2001:~# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.11", GitCommit:"c6a2f08fc4378c5381dd948d9ad9d1080e3e6b33", GitTreeState:"clean", BuildDate:"2021-05-12T12:25:01Z", GoVersion:"go1.15.12", Compiler:"gc", Platform:"linux/amd64"}

```
<code>kubeadm upgrade plan</code>
```
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.19.11
[upgrade/versions] kubeadm version: v1.19.11
I0601 17:13:07.026601 1534531 version.go:255] remote version is much newer: v1.21.1; falling back to: stable-1.19
[upgrade/versions] Latest stable version: v1.19.11
[upgrade/versions] Latest stable version: v1.19.11
[upgrade/versions] Latest version in the v1.19 series: v1.19.11
[upgrade/versions] Latest version in the v1.19 series: v1.19.11
```

<code>kubeadm upgrade apply v1.19.11</code>
```
root@kubemaster2001:~# 
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.19.11"
[upgrade/versions] Cluster version: v1.19.11
[upgrade/versions] kubeadm version: v1.19.11
[upgrade/confirm] Are you sure you want to proceed with the upgrade? [y/N]: y
[upgrade/prepull] Pulling images required for setting up a Kubernetes cluster
[upgrade/prepull] This might take a minute or two, depending on the speed of your internet connection
[upgrade/prepull] You can also perform this action in beforehand using 'kubeadm config images pull'
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.19.11"...
Static pod: kube-apiserver-kubemaster2001 hash: 664573ed7a23be9826c4ecc1d084bd42
Static pod: kube-controller-manager-kubemaster2001 hash: 32c0eef59a861d4801e9bf07b270a032
Static pod: kube-scheduler-kubemaster2001 hash: 6d56e89701adc313ecbc93cdff46d6e6
[upgrade/etcd] Upgrading to TLS for etcd
[upgrade/etcd] Non fatal issue encountered during upgrade: the desired etcd version "3.4.13-0" is not newer than the currently installed "3.4.13-0". Skipping etcd upgrade
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests326424714"
[upgrade/staticpods] Preparing for "kube-apiserver" upgrade
[upgrade/staticpods] Current and new manifests of kube-apiserver are equal, skipping upgrade
[upgrade/staticpods] Preparing for "kube-controller-manager" upgrade
[upgrade/staticpods] Current and new manifests of kube-controller-manager are equal, skipping upgrade
[upgrade/staticpods] Preparing for "kube-scheduler" upgrade
[upgrade/staticpods] Current and new manifests of kube-scheduler are equal, skipping upgrade
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.19" in namespace kube-system with the configuration for the kubelets in the cluster
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.19.11". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
```

<code> 
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.19.11-00 kubectl=1.19.11-00 && \
apt-mark hold kubelet kubectl
</code>


```
Canceled hold on kubelet.
Canceled hold on kubectl.
Get:1 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Hit:3 http://archive.ubuntu.com/ubuntu focal InRelease
Get:4 http://archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Hit:2 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Get:5 http://archive.ubuntu.com/ubuntu focal-backports InRelease [101 kB]
Fetched 328 kB in 7s (44.6 kB/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be upgraded:
  kubectl kubelet
2 upgraded, 0 newly installed, 0 to remove and 7 not upgraded.
Need to get 18.3 MB/26.6 MB of archives.
After this operation, 7168 B disk space will be freed.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.19.11-00 [18.3 MB]
Fetched 18.3 MB in 3s (6402 kB/s)
(Reading database ... 130213 files and directories currently installed.)
Preparing to unpack .../kubectl_1.19.11-00_amd64.deb ...
Unpacking kubectl (1.19.11-00) over (1.19.1-00) ...
Preparing to unpack .../kubelet_1.19.11-00_amd64.deb ...
Unpacking kubelet (1.19.11-00) over (1.19.1-00) ...
Setting up kubectl (1.19.11-00) ...
Setting up kubelet (1.19.11-00) ...
kubelet set on hold.
kubectl set on hold.
```
- Verify the version of the node : <code>kubectl get nodes </code>
```
thinny81@kubemaster2001:~$ kubectl get nodes
NAME             STATUS   ROLES    AGE    VERSION
kubemaster2001   Ready    master   2d8h   v1.19.11
kubeworker2002   Ready    <none>   2d8h   v1.19.1
```
### 2. Upgrade on Worker Nodes
We will follow the same process again as Master Node.
- First upgrade Kubeadm
> apt-get update && apt-get install -y kubeadm=1.19.11-00 && \
> apt-mark hold kubeadm

```
Canceled hold on kubeadm.
Get:2 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Hit:3 http://archive.ubuntu.com/ubuntu focal InRelease
Get:4 http://archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Hit:1 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Get:5 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [669 kB]
Get:6 http://security.ubuntu.com/ubuntu focal-security/main Translation-en [136 kB]
Get:7 http://security.ubuntu.com/ubuntu focal-security/main amd64 c-n-f Metadata [7760 B]
Get:8 http://security.ubuntu.com/ubuntu focal-security/universe amd64 Packages [588 kB]
Get:9 http://archive.ubuntu.com/ubuntu focal-backports InRelease [101 kB]
Get:10 http://security.ubuntu.com/ubuntu focal-security/universe Translation-en [94.4 kB]
Get:11 http://security.ubuntu.com/ubuntu focal-security/universe amd64 c-n-f Metadata [11.5 kB]
Get:12 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages [991 kB]
Get:13 http://archive.ubuntu.com/ubuntu focal-updates/main Translation-en [224 kB]
Get:14 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 c-n-f Metadata [13.4 kB]
Get:15 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 Packages [778 kB]
Get:16 http://archive.ubuntu.com/ubuntu focal-updates/universe Translation-en [167 kB]
Get:17 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 c-n-f Metadata [17.5 kB]
Fetched 4025 kB in 2s (1737 kB/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 8 not upgraded.
Need to get 7758 kB of archives.
After this operation, 49.2 kB disk space will be freed.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.19.11-00 [7758 kB]
Fetched 7758 kB in 8s (928 kB/s)
(Reading database ... 161255 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.19.11-00_amd64.deb ...
Unpacking kubeadm (1.19.11-00) over (1.19.1-00) ...
Setting up kubeadm (1.19.11-00) ...
kubeadm set on hold.
```

<code>root@kubeworker2002:~# kubeadm upgrade node</code>
```
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[preflight] Running pre-flight checks
[preflight] Skipping prepull. Not a control plane node.
[upgrade] Skipping phase. Not a control plane node.
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.
```
- Drain the node to prepare for the upgrade by marking it is unscheduable and evicting the workloads : 
<code>kubectl drain kubeworker2002 --ignore-daemonsets</code>
```
node/kubeworker2002 cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/calico-node-vgd62, kube-system/kube-proxy-pnrsm
evicting pod default/nginx-7848d4b86f-qq7m7
pod/nginx-7848d4b86f-qq7m7 evicted
node/kubeworker2002 evicted

Kubectl output:
================
thinny81@kubemaster2001:~$ kubectl get nodes
NAME             STATUS                     ROLES                  AGE    VERSION
kubemaster2001   Ready                      control-plane,master   2d9h   v1.20.7
kubeworker2002   Ready,SchedulingDisabled   <none>                 2d8h   v1.19.11

```
- Upgrade Kubelet and Kubectl
```
apt-mark unhold kubelet kubectl && \ 
apt-get update && apt-get install -y kubelet=1.19.11-00 kubectl=1.19.11-00 && \
apt-mark hold kubelet kubectl

Output:
=========
Hit:2 http://security.ubuntu.com/ubuntu focal-security InRelease
Hit:3 http://archive.ubuntu.com/ubuntu focal InRelease
Hit:1 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Hit:4 http://archive.ubuntu.com/ubuntu focal-updates InRelease
Hit:5 http://archive.ubuntu.com/ubuntu focal-backports InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be upgraded:
  kubectl kubelet
2 upgraded, 0 newly installed, 0 to remove and 7 not upgraded.
Need to get 26.6 MB of archives.
After this operation, 7168 B disk space will be freed.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubectl amd64 1.19.11-00 [8349 kB]
Get:2 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.19.11-00 [18.3 MB]
Fetched 26.6 MB in 10s (2740 kB/s)
(Reading database ... 161255 files and directories currently installed.)
Preparing to unpack .../kubectl_1.19.11-00_amd64.deb ...
Unpacking kubectl (1.19.11-00) over (1.19.1-00) ...
Preparing to unpack .../kubelet_1.19.11-00_amd64.deb ...
Unpacking kubelet (1.19.11-00) over (1.19.1-00) ...
Setting up kubectl (1.19.11-00) ...
Setting up kubelet (1.19.11-00) ...
kubelet set on hold.
kubectl set on hold.
```

- Restart Kubelet, Daemon and Uncordon the node (to bing back the node online and mark it is schedulable)
```
systemctl daemon-reload
systemctl restart kubelet

kubectl uncordon kubeworker2002
```
- Verify the cluster : 
```
thinny81@kubemaster2001:~$ kubectl get nodes
NAME             STATUS   ROLES    AGE    VERSION
kubemaster2001   Ready    master   2d8h   v1.19.11
kubeworker2002   Ready    <none>   2d8h   v1.19.11
thinny81@kubemaster2001:~$
```

#### We will need to repeat the same steps to bring the cluster to 1.20.x version.
