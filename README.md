# kubernetes-cluster-setup-using-kubeadm
How to create kubernetes cluster using kubeadm

Watch the video :https://youtu.be/cAZ5nkLfL6M

Follow the below steps for creating Kubernetes Cluster on CentOs.

Steps Involved:

1. Set Hostnames
2. Assign Static IP
3. Edit /etc/hosts file
4. Disable SELinux
5. Disable firewall and edit Iptables settings
6. Setup Kubernetes Repo
7. Installing Kubeadm and Docker, Enable and start the services
8. Disable Swap
9. Initialize Kubernetes Cluster
10. Installing Pod Network using Calico network
11. Join Worker Nodes

Steps 1 tp 8 is done on Both Master and worker nodes, Steps 9 & 10 is to be done only on master node, step 11 is done only on worker nodes.

-------------------------------------------------------

**Set Hostnames**

hostnamectl set-hostname k8smaster (On Master)<br />
hostnamectl set-hostname k8sworker1(On Node1)<br />
hostnamectl set-hostname k8sworker2 (On Node2)<br />

------------------------------------------------------

**Assign Static IP**

Run nmcli con to indentify the network details.<br />

Go to vi /etc/sysconfig/network-scripts/ and change the settings in ifcfg-ens33 (the name will change based on your network device name) as below. <br />

Below is a sample format <br />

TYPE="Ethernet"<br />
PROXY_METHOD="none"<br />
BROWSER_ONLY="no"<br />
BOOTPROTO="none"<br />
IPADDR=XXX.XXX.XXX.XXX<br />
PREFIX=24<br />
GATEWAY=XXX.XXX.XXX.XXX<br />
DNS1=192.168.2.254<br />
DNS2=8.8.8.8<br />
DNS3=8.8.4.4<br />
DEFROUTE="yes"<br />
IPV4_FAILURE_FATAL="no"<br />
IPV6INIT="no"<br />
IPV6_AUTOCONF="no"<br />
IPV6_DEFROUTE="no"<br />
IPV6_FAILURE_FATAL="no"<br />
IPV6_ADDR_GEN_MODE="stable-privacy"<br />
NAME="ens33"<br />
UUID="Your respective network UUID"<br />
DEVICE="ens33"<br />
ONBOOT="yes"<br />

Run the command  systemctl restart network to restart the network<br />

------------------------------------------------------------

**Edit /etc/hosts file**

Run the below commands on the machines. Change the IP address and host name as per your machine settings.<br />
cat << EOF >> /etc/hosts<br />

192.168.0.xxx k8smaster<br />
192.168.0.xxx k8sworker1<br />
192.168.0.xxx k8sworker2<br />

EOF
  
-----------------------------------------------------------
  
**Disable SELinux**
  
setenforce 0<br />
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux<br />

-----------------------------------------------------------
  
**Disable firewall and edit Iptables settings**
  
systemctl disable firewalld<br />
modprobe br_netfilter<br />
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables<br />
  
----------------------------------------------------------
  
**Setup Kubernetes Repo**

cat << EOF > /etc/yum.repos.d/kubernetes.repo<br />
[kubernetes]<br />
name=Kubernetes<br />
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64<br />
enabled=1<br />
gpgcheck=1<br />
repo_gpgcheck=1<br />
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg<br />
EOF<br />

---------------------------------------------------------
  
**Installing Kubeadm and Docker, Enable and start the services**
  
yum install kubeadm docker -y<br />
systemctl enable kubelet<br />
systemctl start kubelet<br />
systemctl enable docker<br />
systemctl start docker<br />
  
--------------------------------------------------------
  
**Disable Swap**
  
swapoff -a<br />
vi /etc/fstab and Comment the line with Swap Keyword<br />
  
-----------------------------------------------------
  
**Initialize Kubernetes Cluster**

kubeadm init<br />
mkdir -p $HOME/.kube<br />
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config<br />
chown $(id -u):$(id -g) $HOME/.kube/config<br />
  
------------------------------------------------------
  
**Installing Pod Network using Calico network**


curl https://docs.projectcalico.org/manifests/calico.yaml -O<br />
kubectl apply -f calico.yaml<br />
kubectl get pods -n kube-system<br />

----------------------------------------------------

  **Join Worker Nodes **
  
  Use the token from Kubeadmin init screen. Below is a sample how it looks like.<br />
  
  kubeadm join 192.168.0.xxx:6443 --token XXX\
        --discovery-token-ca-cert-hash sha256:XX<br />

  ---------------------------------------------------
  
  
  
  
  ---------------------Putty history -----------------------
  
  
Kubernetes

Day-1:


 1) Introduction
 2) Kubernetes Arch/Services introduction
 3) lab Setup
 4) Installation



Kubernetes: it is an  opensource container orchestration and managment tools.



1) centerilized managment : 
               1- we are able to manage number of container machines from single site.
               2- we can create/edit/update container  life .
               3- also maintaing container state as well.


2)  Cluster Mode :
         1) now container container system are working together 
         2) now failover is possible between container machines.
         3) Highavailability

3) Scalability

            1) On demand  you can add/removed container in running application
            2) Manually
            3) automatically: based on cpu/memory/network used container should  be added automatically.

4) Rollout and Rollback:
             1)  on zero down time we can upgrade  application x version to y version.

_____________________________________________________________________________________________





Cluster Arch and service Introduction: 




1) Kubernetes Master Node: 


       1) kube-apiserver: it does validation (authencation + autherization) of any instruction which given  by administrator 
       2) kube-etcd: it contains persistent information for API. all information  about the cluster it write inside the etcd . API only has permission to write the etcd.
       3) kube-scheduler: it provide worker Node selection/choose for your container workload 
       4) kube-controller: it provide api instruction to worker Nodes.  it maintain commuication with worker Nodes.



User--API--etcd---scheduler---Controller----goes to workerNodeside---- kubelet-Containeruntime--Container

1.24

2) Worker Nodes: 


   1) Container Runtime:

                 1) docker community:
                              1) dockerd
                              2) containerd 

                2) Cri-io

  2)  kubelet:  it is agent of  kubernete Master which is running on all worker Nodes.
                 it basically recieve instruction from kube-controller then apply on container runtime.


  3)  kube-proxy: it is network related service which controll network traffic for containers.





Lab Setup:


3 machines 


Master
2GB
1cpu
10HD


worker-1
2GB
1 cpu
10HD

worker-2

2GB
1cpu
10HD

Centos:7


1) Local laptop
2) Cloud Based

_____________________________________

Cloud Service Model:

1) IAAS
2) PAAS:
             Aws: EKS
             google: GKE
             azure: AKS
   
3) SAAS



How to  install Kubernetes Master and worker Nodes.




  Mode :

     1)  control plane Mode: 




  1) container runtime(containerd/cri-io)

  2) kube-proxy

  3) kubelet

7 services:


Systemd Mode

container run time
kubelet

Control plane:

api
etcd
controller
scheduler
kube-proxy








  control plane  installation;

       1) kubeadm:  it control plane installation utility

        2) KOPS

       3) ansible



change hostname and resolv by /etc/hosts files
Selinux should be disabled only on master Nodes
Swap partition should be disable over all nodes.
step1: kubeadm 
step2: container run time
step3: kubelet
step4: you will start  4 service  ( api/scheduler/etcd/controller)


worker Niode:

1) Contatiner runtime
2) kubelet





 2) systemd Mode: Recording 





kubeadm: Installation utility
kubectl: command utiliy : only required master nodes.

kubelet: agent of kubernetes master 





[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]# ifconfig eth0
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 9001
        inet 172.31.45.35  netmask 255.255.240.0  broadcast 172.31.47.255
        inet6 fe80::13:46ff:fee5:ae52  prefixlen 64  scopeid 0x20<link>
        ether 02:13:46:e5:ae:52  txqueuelen 1000  (Ethernet)
        RX packets 1076  bytes 95947 (93.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 932  bytes 108443 (105.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@kubemaster ~]# vi /etc/hosts
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]# ping kubemaster
PING kubemaster (172.31.45.35) 56(84) bytes of data.
64 bytes from kubemaster (172.31.45.35): icmp_seq=1 ttl=64 time=0.019 ms
64 bytes from kubemaster (172.31.45.35): icmp_seq=2 ttl=64 time=0.027 ms
64 bytes from kubemaster (172.31.45.35): icmp_seq=3 ttl=64 time=0.028 ms
64 bytes from kubemaster (172.31.45.35): icmp_seq=4 ttl=64 time=0.028 ms
^C
--- kubemaster ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2999ms
rtt min/avg/max/mdev = 0.019/0.025/0.028/0.006 ms
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]# clear
[root@kubemaster ~]#
[root@kubemaster ~]# getenforce
Enforcing
[root@kubemaster ~]# setenforce 0
[root@kubemaster ~]#
[root@kubemaster ~]# getenforce
Permissive
[root@kubemaster ~]# vi /etc/sysconfig/selinux
[root@kubemaster ~]#
[root@kubemaster ~]# swapon -s
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]# vi /etc/yum.repos.d/kube.repo
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]# yum install kubeadm -y
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: download.cf.centos.org
 * extras: download.cf.centos.org
 * updates: download.cf.centos.org
base                                                                  | 3.6 kB  00:00:00
extras                                                                | 2.9 kB  00:00:00
kubernetes                                                            | 1.4 kB  00:00:00
updates                                                               | 2.9 kB  00:00:00
(1/5): base/7/x86_64/group_gz                                         | 153 kB  00:00:00
(2/5): extras/7/x86_64/primary_db                                     | 247 kB  00:00:00
(4/5): kubernetes/x86_64/primary 3% [-                     ]  0.0 B/s | 821 kB  --:--:-- ETA
(3/5): kubernetes/x86_64/primary                                      | 112 kB  00:00:01
(4/5): updates/7/x86_64/primary_db                                    |  16 MB  00:00:01
(5/5): base/7/x86_64/primary_db                                       | 6.1 MB  00:00:01


kubernetes                                                                           832/832
Resolving Dependencies
--> Running transaction check
---> Package kubeadm.x86_64 0:1.24.3-0 will be installed
--> Processing Dependency: kubernetes-cni >= 0.8.6 for package: kubeadm-1.24.3-0.x86_64
--> Processing Dependency: kubelet >= 1.19.0 for package: kubeadm-1.24.3-0.x86_64
--> Processing Dependency: kubectl >= 1.19.0 for package: kubeadm-1.24.3-0.x86_64
--> Processing Dependency: cri-tools >= 1.19.0 for package: kubeadm-1.24.3-0.x86_64
--> Running transaction check
---> Package cri-tools.x86_64 0:1.24.2-0 will be installed
---> Package kubectl.x86_64 0:1.24.3-0 will be installed
---> Package kubelet.x86_64 0:1.24.3-0 will be installed
--> Processing Dependency: socat for package: kubelet-1.24.3-0.x86_64
--> Processing Dependency: ebtables for package: kubelet-1.24.3-0.x86_64
--> Processing Dependency: conntrack for package: kubelet-1.24.3-0.x86_64
---> Package kubernetes-cni.x86_64 0:0.8.7-0 will be installed
--> Running transaction check
---> Package conntrack-tools.x86_64 0:1.4.4-7.el7 will be installed
--> Processing Dependency: libnetfilter_cttimeout.so.1(LIBNETFILTER_CTTIMEOUT_1.1)(64bit) for                                package: conntrack-tools-1.4.4-7.el7.x86_64
--> Processing Dependency: libnetfilter_cttimeout.so.1(LIBNETFILTER_CTTIMEOUT_1.0)(64bit) for                                package: conntrack-tools-1.4.4-7.el7.x86_64
--> Processing Dependency: libnetfilter_cthelper.so.0(LIBNETFILTER_CTHELPER_1.0)(64bit) for p                               ackage: conntrack-tools-1.4.4-7.el7.x86_64
--> Processing Dependency: libnetfilter_queue.so.1()(64bit) for package: conntrack-tools-1.4.                               4-7.el7.x86_64
--> Processing Dependency: libnetfilter_cttimeout.so.1()(64bit) for package: conntrack-tools-                               1.4.4-7.el7.x86_64
--> Processing Dependency: libnetfilter_cthelper.so.0()(64bit) for package: conntrack-tools-1                               .4.4-7.el7.x86_64
---> Package ebtables.x86_64 0:2.0.10-16.el7 will be installed
---> Package socat.x86_64 0:1.7.3.2-2.el7 will be installed
--> Running transaction check
---> Package libnetfilter_cthelper.x86_64 0:1.0.0-11.el7 will be installed
---> Package libnetfilter_cttimeout.x86_64 0:1.0.0-7.el7 will be installed
---> Package libnetfilter_queue.x86_64 0:1.0.2-2.el7_2 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=============================================================================================
 Package                       Arch          Version                 Repository         Size
=============================================================================================
Installing:
 kubeadm                       x86_64        1.24.3-0                kubernetes        9.5 M
Installing for dependencies:
 conntrack-tools               x86_64        1.4.4-7.el7             base              187 k
 cri-tools                     x86_64        1.24.2-0                kubernetes        5.9 M
 ebtables                      x86_64        2.0.10-16.el7           base              123 k
 kubectl                       x86_64        1.24.3-0                kubernetes        9.9 M
 kubelet                       x86_64        1.24.3-0                kubernetes         20 M
 kubernetes-cni                x86_64        0.8.7-0                 kubernetes         19 M
 libnetfilter_cthelper         x86_64        1.0.0-11.el7            base               18 k
 libnetfilter_cttimeout        x86_64        1.0.0-7.el7             base               18 k
 libnetfilter_queue            x86_64        1.0.2-2.el7_2           base               23 k
 socat                         x86_64        1.7.3.2-2.el7           base              290 k

Transaction Summary
=============================================================================================
Install  1 Package (+10 Dependent packages)

Total download size: 65 M
Installed size: 279 M
Downloading packages:
(1/11): ebtables-2.0.10-16.el7.x86_64.rpm                             | 123 kB  00:00:00
(2/11): conntrack-tools-1.4.4-7.el7.x86_64.rpm                        | 187 kB  00:00:00
(3/11): 07433570e95a2782cc127e659fe6df434db7f88805e2aed6067768d2f32cb | 5.9 MB  00:00:00
(4/11): ae095661b772845e86a4283d12077069d751176dac90a9046288846645fc6 | 9.9 MB  00:00:00
(5/11): f2c23e99b3c2b1f42b77c7fae9ca6a324714640460ae910f0a763673dc941 | 9.5 MB  00:00:01
(6/11): libnetfilter_cthelper-1.0.0-11.el7.x86_64.rpm                 |  18 kB  00:00:00
(7/11): libnetfilter_queue-1.0.2-2.el7_2.x86_64.rpm                   |  23 kB  00:00:00
(8/11): socat-1.7.3.2-2.el7.x86_64.rpm                                | 290 kB  00:00:00
(9/11): libnetfilter_cttimeout-1.0.0-7.el7.x86_64.rpm                 |  18 kB  00:00:00
(10/11): db7cb5cb0b3f6875f54d10f02e625573988e3e91fd4fc5eef0b1876bb186 |  19 MB  00:00:01
(11/11): 7cb2519fffeb4dcd35c4c8b0ae9f1a049c06a8d495cb60334d4f46a692cc |  20 MB  00:00:02
---------------------------------------------------------------------------------------------
Total                                                         17 MB/s |  65 MB  00:00:03
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : ebtables-2.0.10-16.el7.x86_64                                            1/11
  Installing : libnetfilter_cttimeout-1.0.0-7.el7.x86_64                                2/11
  Installing : socat-1.7.3.2-2.el7.x86_64                                               3/11
  Installing : cri-tools-1.24.2-0.x86_64                                                4/11
  Installing : kubectl-1.24.3-0.x86_64                                                  5/11
  Installing : libnetfilter_queue-1.0.2-2.el7_2.x86_64                                  6/11
  Installing : libnetfilter_cthelper-1.0.0-11.el7.x86_64                                7/11
  Installing : conntrack-tools-1.4.4-7.el7.x86_64                                       8/11
  Installing : kubernetes-cni-0.8.7-0.x86_64                                            9/11
  Installing : kubelet-1.24.3-0.x86_64                                                 10/11
  Installing : kubeadm-1.24.3-0.x86_64                                                 11/11
  Verifying  : kubelet-1.24.3-0.x86_64                                                  1/11
  Verifying  : libnetfilter_cthelper-1.0.0-11.el7.x86_64                                2/11
  Verifying  : conntrack-tools-1.4.4-7.el7.x86_64                                       3/11
  Verifying  : kubeadm-1.24.3-0.x86_64                                                  4/11
  Verifying  : kubernetes-cni-0.8.7-0.x86_64                                            5/11
  Verifying  : libnetfilter_queue-1.0.2-2.el7_2.x86_64                                  6/11
  Verifying  : kubectl-1.24.3-0.x86_64                                                  7/11
  Verifying  : cri-tools-1.24.2-0.x86_64                                                8/11
  Verifying  : socat-1.7.3.2-2.el7.x86_64                                               9/11
  Verifying  : libnetfilter_cttimeout-1.0.0-7.el7.x86_64                               10/11
  Verifying  : ebtables-2.0.10-16.el7.x86_64                                           11/11

Installed:
  kubeadm.x86_64 0:1.24.3-0

Dependency Installed:
  conntrack-tools.x86_64 0:1.4.4-7.el7          cri-tools.x86_64 0:1.24.2-0
  ebtables.x86_64 0:2.0.10-16.el7               kubectl.x86_64 0:1.24.3-0
  kubelet.x86_64 0:1.24.3-0                     kubernetes-cni.x86_64 0:0.8.7-0
  libnetfilter_cthelper.x86_64 0:1.0.0-11.el7   libnetfilter_cttimeout.x86_64 0:1.0.0-7.el7
  libnetfilter_queue.x86_64 0:1.0.2-2.el7_2     socat.x86_64 0:1.7.3.2-2.el7

Complete!
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]# yum-config-manager \
>     --add-repo \
>     https://download.docker.com/linux/centos/docker-ce.repo
Loaded plugins: fastestmirror
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/doc                               ker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]# yum install docker-ce -y
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: download.cf.centos.org
 * extras: download.cf.centos.org
 * updates: download.cf.centos.org
docker-ce-stable                                                      | 3.5 kB  00:00:00
(1/2): docker-ce-stable/7/x86_64/primary_db                           |  80 kB  00:00:00
(2/2): docker-ce-stable/7/x86_64/updateinfo                           |   55 B  00:00:00
Resolving Dependencies
--> Running transaction check
---> Package docker-ce.x86_64 3:20.10.17-3.el7 will be installed
--> Processing Dependency: container-selinux >= 2:2.74 for package: 3:docker-ce-20.10.17-3.el                               7.x86_64
--> Processing Dependency: containerd.io >= 1.4.1 for package: 3:docker-ce-20.10.17-3.el7.x86                               _64
--> Processing Dependency: docker-ce-cli for package: 3:docker-ce-20.10.17-3.el7.x86_64
--> Processing Dependency: docker-ce-rootless-extras for package: 3:docker-ce-20.10.17-3.el7.                               x86_64
--> Running transaction check
---> Package container-selinux.noarch 2:2.119.2-1.911c772.el7_8 will be installed
---> Package containerd.io.x86_64 0:1.6.7-3.1.el7 will be installed
---> Package docker-ce-cli.x86_64 1:20.10.17-3.el7 will be installed
--> Processing Dependency: docker-scan-plugin(x86-64) for package: 1:docker-ce-cli-20.10.17-3                               .el7.x86_64
---> Package docker-ce-rootless-extras.x86_64 0:20.10.17-3.el7 will be installed
--> Processing Dependency: fuse-overlayfs >= 0.7 for package: docker-ce-rootless-extras-20.10                               .17-3.el7.x86_64
--> Processing Dependency: slirp4netns >= 0.4 for package: docker-ce-rootless-extras-20.10.17                               -3.el7.x86_64
--> Running transaction check
---> Package docker-scan-plugin.x86_64 0:0.17.0-3.el7 will be installed
---> Package fuse-overlayfs.x86_64 0:0.7.2-6.el7_8 will be installed
--> Processing Dependency: libfuse3.so.3(FUSE_3.2)(64bit) for package: fuse-overlayfs-0.7.2-6                               .el7_8.x86_64
--> Processing Dependency: libfuse3.so.3(FUSE_3.0)(64bit) for package: fuse-overlayfs-0.7.2-6                               .el7_8.x86_64
--> Processing Dependency: libfuse3.so.3()(64bit) for package: fuse-overlayfs-0.7.2-6.el7_8.x                               86_64
---> Package slirp4netns.x86_64 0:0.4.3-4.el7_8 will be installed
--> Running transaction check
---> Package fuse3-libs.x86_64 0:3.6.1-4.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=============================================================================================
 Package                     Arch     Version                       Repository          Size
=============================================================================================
Installing:
 docker-ce                   x86_64   3:20.10.17-3.el7              docker-ce-stable    22 M
Installing for dependencies:
 container-selinux           noarch   2:2.119.2-1.911c772.el7_8     extras              40 k
 containerd.io               x86_64   1.6.7-3.1.el7                 docker-ce-stable    33 M
 docker-ce-cli               x86_64   1:20.10.17-3.el7              docker-ce-stable    29 M
 docker-ce-rootless-extras   x86_64   20.10.17-3.el7                docker-ce-stable   8.2 M
 docker-scan-plugin          x86_64   0.17.0-3.el7                  docker-ce-stable   3.7 M
 fuse-overlayfs              x86_64   0.7.2-6.el7_8                 extras              54 k
 fuse3-libs                  x86_64   3.6.1-4.el7                   extras              82 k
 slirp4netns                 x86_64   0.4.3-4.el7_8                 extras              81 k

Transaction Summary
=============================================================================================
Install  1 Package (+8 Dependent packages)

Total download size: 97 M
Installed size: 394 M
Downloading packages:
(1/9): container-selinux-2.119.2-1.911c772.el7_8.noarch.rpm           |  40 kB  00:00:00
warning: /var/cache/yum/x86_64/7/docker-ce-stable/packages/docker-ce-20.10.17-3.el7.x86_64.rp                               m: Header V4 RSA/SHA512 Signature, key ID 621e9f35: NOKEY
Public key for docker-ce-20.10.17-3.el7.x86_64.rpm is not installed
(2/9): docker-ce-20.10.17-3.el7.x86_64.rpm                            |  22 MB  00:00:00
(3/9): containerd.io-1.6.7-3.1.el7.x86_64.rpm                         |  33 MB  00:00:01
(4/9): docker-ce-cli-20.10.17-3.el7.x86_64.rpm                        |  29 MB  00:00:00
(5/9): docker-ce-rootless-extras-20.10.17-3.el7.x86_64.rpm            | 8.2 MB  00:00:00
(6/9): fuse-overlayfs-0.7.2-6.el7_8.x86_64.rpm                        |  54 kB  00:00:00
(7/9): docker-scan-plugin-0.17.0-3.el7.x86_64.rpm                     | 3.7 MB  00:00:00
(8/9): slirp4netns-0.4.3-4.el7_8.x86_64.rpm                           |  81 kB  00:00:00
(9/9): fuse3-libs-3.6.1-4.el7.x86_64.rpm                              |  82 kB  00:00:00
---------------------------------------------------------------------------------------------
Total                                                         55 MB/s |  97 MB  00:00:01
Retrieving key from https://download.docker.com/linux/centos/gpg
Importing GPG key 0x621E9F35:
 Userid     : "Docker Release (CE rpm) <docker@docker.com>"
 Fingerprint: 060a 61c5 1b55 8a7f 742b 77aa c52f eb6b 621e 9f35
 From       : https://download.docker.com/linux/centos/gpg
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : docker-scan-plugin-0.17.0-3.el7.x86_64                                    1/9
  Installing : 1:docker-ce-cli-20.10.17-3.el7.x86_64                                     2/9
  Installing : 2:container-selinux-2.119.2-1.911c772.el7_8.noarch                        3/9
  Installing : containerd.io-1.6.7-3.1.el7.x86_64                                        4/9
  Installing : slirp4netns-0.4.3-4.el7_8.x86_64                                          5/9
  Installing : fuse3-libs-3.6.1-4.el7.x86_64                                             6/9
  Installing : fuse-overlayfs-0.7.2-6.el7_8.x86_64                                       7/9
  Installing : 3:docker-ce-20.10.17-3.el7.x86_64                                         8/9
  Installing : docker-ce-rootless-extras-20.10.17-3.el7.x86_64                           9/9
  Verifying  : fuse3-libs-3.6.1-4.el7.x86_64                                             1/9
  Verifying  : docker-ce-rootless-extras-20.10.17-3.el7.x86_64                           2/9
  Verifying  : 1:docker-ce-cli-20.10.17-3.el7.x86_64                                     3/9
  Verifying  : containerd.io-1.6.7-3.1.el7.x86_64                                        4/9
  Verifying  : slirp4netns-0.4.3-4.el7_8.x86_64                                          5/9
  Verifying  : 2:container-selinux-2.119.2-1.911c772.el7_8.noarch                        6/9
  Verifying  : 3:docker-ce-20.10.17-3.el7.x86_64                                         7/9
  Verifying  : docker-scan-plugin-0.17.0-3.el7.x86_64                                    8/9
  Verifying  : fuse-overlayfs-0.7.2-6.el7_8.x86_64                                       9/9

Installed:
  docker-ce.x86_64 3:20.10.17-3.el7

Dependency Installed:
  container-selinux.noarch 2:2.119.2-1.911c772.el7_8
  containerd.io.x86_64 0:1.6.7-3.1.el7
  docker-ce-cli.x86_64 1:20.10.17-3.el7
  docker-ce-rootless-extras.x86_64 0:20.10.17-3.el7
  docker-scan-plugin.x86_64 0:0.17.0-3.el7
  fuse-overlayfs.x86_64 0:0.7.2-6.el7_8
  fuse3-libs.x86_64 0:3.6.1-4.el7
  slirp4netns.x86_64 0:0.4.3-4.el7_8

Complete!
[root@kubemaster ~]# \
> ^C
[root@kubemaster ~]# systemctl start  docker
[root@kubemaster ~]# systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/s                               ystemd/system/docker.service.
[root@kubemaster ~]#
[root@kubemaster ~]# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 05:42 ?        00:00:04 /usr/lib/systemd/systemd --switched-root --sy
root         2     0  0 05:42 ?        00:00:00 [kthreadd]
root         4     2  0 05:42 ?        00:00:00 [kworker/0:0H]
root         5     2  0 05:42 ?        00:00:00 [kworker/u30:0]
root         6     2  0 05:42 ?        00:00:00 [ksoftirqd/0]
root         7     2  0 05:42 ?        00:00:00 [migration/0]
root         8     2  0 05:42 ?        00:00:00 [rcu_bh]
root         9     2  0 05:42 ?        00:00:00 [rcu_sched]
root        10     2  0 05:42 ?        00:00:00 [lru-add-drain]
root        11     2  0 05:42 ?        00:00:00 [watchdog/0]
root        13     2  0 05:42 ?        00:00:00 [kdevtmpfs]
root        14     2  0 05:42 ?        00:00:00 [netns]
root        15     2  0 05:42 ?        00:00:00 [xenwatch]
root        16     2  0 05:42 ?        00:00:00 [xenbus]
root        18     2  0 05:42 ?        00:00:00 [khungtaskd]
root        19     2  0 05:42 ?        00:00:00 [writeback]
root        20     2  0 05:42 ?        00:00:00 [kintegrityd]
root        21     2  0 05:42 ?        00:00:00 [bioset]
root        22     2  0 05:42 ?        00:00:00 [bioset]
root        23     2  0 05:42 ?        00:00:00 [bioset]
root        24     2  0 05:42 ?        00:00:00 [kblockd]
root        25     2  0 05:42 ?        00:00:00 [md]
root        26     2  0 05:42 ?        00:00:00 [edac-poller]
root        27     2  0 05:42 ?        00:00:00 [watchdogd]
root        32     2  0 05:42 ?        00:00:00 [kswapd0]
root        33     2  0 05:42 ?        00:00:00 [ksmd]
root        34     2  0 05:42 ?        00:00:00 [khugepaged]
root        35     2  0 05:42 ?        00:00:00 [crypto]
root        43     2  0 05:42 ?        00:00:00 [kthrotld]
root        44     2  0 05:42 ?        00:00:00 [kworker/u30:1]
root        45     2  0 05:42 ?        00:00:00 [kmpath_rdacd]
root        46     2  0 05:42 ?        00:00:00 [kaluad]
root        47     2  0 05:42 ?        00:00:00 [kpsmoused]
root        49     2  0 05:42 ?        00:00:00 [ipv6_addrconf]
root        62     2  0 05:42 ?        00:00:00 [deferwq]
root       122     2  0 05:42 ?        00:00:00 [kauditd]
root       182     2  0 05:42 ?        00:00:00 [rpciod]
root       183     2  0 05:42 ?        00:00:00 [xprtiod]
root       249     2  0 05:42 ?        00:00:00 [ata_sff]
root       254     2  0 05:42 ?        00:00:00 [scsi_eh_0]
root       256     2  0 05:42 ?        00:00:00 [scsi_tmf_0]
root       257     2  0 05:42 ?        00:00:00 [scsi_eh_1]
root       258     2  0 05:42 ?        00:00:00 [scsi_tmf_1]
root       270     2  0 05:42 ?        00:00:00 [bioset]
root       271     2  0 05:42 ?        00:00:00 [xfsalloc]
root       272     2  0 05:42 ?        00:00:00 [xfs_mru_cache]
root       273     2  0 05:42 ?        00:00:00 [xfs-buf/xvda1]
root       274     2  0 05:42 ?        00:00:00 [xfs-data/xvda1]
root       275     2  0 05:42 ?        00:00:00 [xfs-conv/xvda1]
root       276     2  0 05:42 ?        00:00:00 [xfs-cil/xvda1]
root       277     2  0 05:42 ?        00:00:00 [xfs-reclaim/xvd]
root       278     2  0 05:42 ?        00:00:00 [xfs-log/xvda1]
root       279     2  0 05:42 ?        00:00:00 [xfs-eofblocks/x]
root       280     2  0 05:42 ?        00:00:00 [xfsaild/xvda1]
root       281     2  0 05:42 ?        00:00:00 [kworker/0:1H]
root       388     1  0 05:42 ?        00:00:00 /usr/lib/systemd/systemd-journald
root       420     1  0 05:42 ?        00:00:00 /usr/lib/systemd/systemd-udevd
root       456     1  0 05:42 ?        00:00:00 /sbin/auditd
rpc        508     1  0 05:42 ?        00:00:00 /sbin/rpcbind -w
dbus       522     1  0 05:42 ?        00:00:00 /usr/bin/dbus-daemon --system --address=syste
root       528     2  0 05:42 ?        00:00:00 [ttm_swap]
chrony     536     1  0 05:42 ?        00:00:00 /usr/sbin/chronyd
root       561     1  0 05:42 ?        00:00:00 /usr/sbin/gssproxy -D
root       571     1  0 05:42 ?        00:00:00 /usr/lib/systemd/systemd-logind
polkitd    574     1  0 05:42 ?        00:00:00 /usr/lib/polkit-1/polkitd --no-debug
root       838     1  0 05:43 ?        00:00:00 /sbin/dhclient -1 -q -lf /var/lib/dhclient/dh
root       895     1  0 05:43 ?        00:00:00 /usr/bin/python2 -Es /usr/sbin/tuned -l -P
root      1025     1  0 05:43 ?        00:00:00 /usr/libexec/postfix/master -w
postfix   1026  1025  0 05:43 ?        00:00:00 pickup -l -t unix -u
postfix   1027  1025  0 05:43 ?        00:00:00 qmgr -l -t unix -u
root      1100     1  0 05:43 ?        00:00:00 /usr/sbin/rsyslogd -n
root      1119     1  0 05:43 ?        00:00:00 /usr/sbin/crond -n
root      1120     1  0 05:43 ttyS0    00:00:00 /sbin/agetty --keep-baud 115200,38400,9600 tt
root      1124     1  0 05:43 tty1     00:00:00 /sbin/agetty --noclear tty1 linux
root      7650     1  0 05:44 ?        00:00:00 /usr/sbin/sshd -D
root      7670  7650  0 05:46 ?        00:00:00 sshd: centos [priv]
centos    7673  7670  0 05:46 ?        00:00:00 sshd: centos@pts/0
centos    7674  7673  0 05:46 pts/0    00:00:00 -bash
root      7693  7674  0 05:46 pts/0    00:00:00 sudo -i
root      7695  7693  0 05:46 pts/0    00:00:00 -bash
root      7714  7695  0 05:47 pts/0    00:00:00 bash
root      7729     2  0 05:47 ?        00:00:00 [kworker/0:0]
root      7736     2  0 05:52 ?        00:00:00 [kworker/0:2]
root      7937     2  0 05:57 ?        00:00:00 [kworker/0:1]
root      7949     1  0 05:57 ?        00:00:00 /usr/bin/containerd
root      7958     1  0 05:57 ?        00:00:00 /usr/bin/dockerd -H fd:// --containerd=/run/c
root      8106  7714  0 05:57 pts/0    00:00:00 ps -ef
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 05:42 ?        00:00:04 /usr/lib/systemd/systemd --switched-root --system --deserialize 21
root         2     0  0 05:42 ?        00:00:00 [kthreadd]
root         4     2  0 05:42 ?        00:00:00 [kworker/0:0H]
root         5     2  0 05:42 ?        00:00:00 [kworker/u30:0]
root         6     2  0 05:42 ?        00:00:00 [ksoftirqd/0]
root         7     2  0 05:42 ?        00:00:00 [migration/0]
root         8     2  0 05:42 ?        00:00:00 [rcu_bh]
root         9     2  0 05:42 ?        00:00:00 [rcu_sched]
root        10     2  0 05:42 ?        00:00:00 [lru-add-drain]
root        11     2  0 05:42 ?        00:00:00 [watchdog/0]
root        13     2  0 05:42 ?        00:00:00 [kdevtmpfs]
root        14     2  0 05:42 ?        00:00:00 [netns]
root        15     2  0 05:42 ?        00:00:00 [xenwatch]
root        16     2  0 05:42 ?        00:00:00 [xenbus]
root        18     2  0 05:42 ?        00:00:00 [khungtaskd]
root        19     2  0 05:42 ?        00:00:00 [writeback]
root        20     2  0 05:42 ?        00:00:00 [kintegrityd]
root        21     2  0 05:42 ?        00:00:00 [bioset]
root        22     2  0 05:42 ?        00:00:00 [bioset]
root        23     2  0 05:42 ?        00:00:00 [bioset]
root        24     2  0 05:42 ?        00:00:00 [kblockd]
root        25     2  0 05:42 ?        00:00:00 [md]
root        26     2  0 05:42 ?        00:00:00 [edac-poller]
root        27     2  0 05:42 ?        00:00:00 [watchdogd]
root        32     2  0 05:42 ?        00:00:00 [kswapd0]
root        33     2  0 05:42 ?        00:00:00 [ksmd]
root        34     2  0 05:42 ?        00:00:00 [khugepaged]
root        35     2  0 05:42 ?        00:00:00 [crypto]
root        43     2  0 05:42 ?        00:00:00 [kthrotld]
root        44     2  0 05:42 ?        00:00:00 [kworker/u30:1]
root        45     2  0 05:42 ?        00:00:00 [kmpath_rdacd]
root        46     2  0 05:42 ?        00:00:00 [kaluad]
root        47     2  0 05:42 ?        00:00:00 [kpsmoused]
root        49     2  0 05:42 ?        00:00:00 [ipv6_addrconf]
root        62     2  0 05:42 ?        00:00:00 [deferwq]
root       122     2  0 05:42 ?        00:00:00 [kauditd]
root       182     2  0 05:42 ?        00:00:00 [rpciod]
root       183     2  0 05:42 ?        00:00:00 [xprtiod]
root       249     2  0 05:42 ?        00:00:00 [ata_sff]
root       254     2  0 05:42 ?        00:00:00 [scsi_eh_0]
root       256     2  0 05:42 ?        00:00:00 [scsi_tmf_0]
root       257     2  0 05:42 ?        00:00:00 [scsi_eh_1]
root       258     2  0 05:42 ?        00:00:00 [scsi_tmf_1]
root       270     2  0 05:42 ?        00:00:00 [bioset]
root       271     2  0 05:42 ?        00:00:00 [xfsalloc]
root       272     2  0 05:42 ?        00:00:00 [xfs_mru_cache]
root       273     2  0 05:42 ?        00:00:00 [xfs-buf/xvda1]
root       274     2  0 05:42 ?        00:00:00 [xfs-data/xvda1]
root       275     2  0 05:42 ?        00:00:00 [xfs-conv/xvda1]
root       276     2  0 05:42 ?        00:00:00 [xfs-cil/xvda1]
root       277     2  0 05:42 ?        00:00:00 [xfs-reclaim/xvd]
root       278     2  0 05:42 ?        00:00:00 [xfs-log/xvda1]
root       279     2  0 05:42 ?        00:00:00 [xfs-eofblocks/x]
root       280     2  0 05:42 ?        00:00:00 [xfsaild/xvda1]
root       281     2  0 05:42 ?        00:00:00 [kworker/0:1H]
root       388     1  0 05:42 ?        00:00:00 /usr/lib/systemd/systemd-journald
root       420     1  0 05:42 ?        00:00:00 /usr/lib/systemd/systemd-udevd
root       456     1  0 05:42 ?        00:00:00 /sbin/auditd
rpc        508     1  0 05:42 ?        00:00:00 /sbin/rpcbind -w
dbus       522     1  0 05:42 ?        00:00:00 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --syst
root       528     2  0 05:42 ?        00:00:00 [ttm_swap]
chrony     536     1  0 05:42 ?        00:00:00 /usr/sbin/chronyd
root       561     1  0 05:42 ?        00:00:00 /usr/sbin/gssproxy -D
root       571     1  0 05:42 ?        00:00:00 /usr/lib/systemd/systemd-logind
polkitd    574     1  0 05:42 ?        00:00:00 /usr/lib/polkit-1/polkitd --no-debug
root       838     1  0 05:43 ?        00:00:00 /sbin/dhclient -1 -q -lf /var/lib/dhclient/dhclient--eth0.lease -pf /var/run
root       895     1  0 05:43 ?        00:00:00 /usr/bin/python2 -Es /usr/sbin/tuned -l -P
root      1025     1  0 05:43 ?        00:00:00 /usr/libexec/postfix/master -w
postfix   1026  1025  0 05:43 ?        00:00:00 pickup -l -t unix -u
postfix   1027  1025  0 05:43 ?        00:00:00 qmgr -l -t unix -u
root      1100     1  0 05:43 ?        00:00:00 /usr/sbin/rsyslogd -n
root      1119     1  0 05:43 ?        00:00:00 /usr/sbin/crond -n
root      1120     1  0 05:43 ttyS0    00:00:00 /sbin/agetty --keep-baud 115200,38400,9600 ttyS0 vt220
root      1124     1  0 05:43 tty1     00:00:00 /sbin/agetty --noclear tty1 linux
root      7650     1  0 05:44 ?        00:00:00 /usr/sbin/sshd -D
root      7670  7650  0 05:46 ?        00:00:00 sshd: centos [priv]
centos    7673  7670  0 05:46 ?        00:00:00 sshd: centos@pts/0
centos    7674  7673  0 05:46 pts/0    00:00:00 -bash
root      7693  7674  0 05:46 pts/0    00:00:00 sudo -i
root      7695  7693  0 05:46 pts/0    00:00:00 -bash
root      7714  7695  0 05:47 pts/0    00:00:00 bash
root      7729     2  0 05:47 ?        00:00:00 [kworker/0:0]
root      7736     2  0 05:52 ?        00:00:00 [kworker/0:2]
root      7937     2  0 05:57 ?        00:00:00 [kworker/0:1]
root      7949     1  0 05:57 ?        00:00:00 /usr/bin/containerd
root      7958     1  0 05:57 ?        00:00:00 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
root      8107  7714  0 05:58 pts/0    00:00:00 ps -ef
[root@kubemaster ~]#
[root@kubemaster ~]# rpm -qa kubelet
kubelet-1.24.3-0.x86_64
[root@kubemaster ~]#
[root@kubemaster ~]# systemctl start kubelet
[root@kubemaster ~]# systemctl enable kubelet
Created symlink from /etc/systemd/system/multi-user.target.wants/kubelet.service to /usr/lib/systemd/system/kubelet.service.
[root@kubemaster ~]#
[root@kubemaster ~]# kubeadm init
[init] Using Kubernetes version: v1.24.3
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR NumCPU]: the number of available CPUs 1 is less than the required 2
        [ERROR CRI]: container runtime is not running: output: E0813 06:00:03.516050    8181 remote_runtime.go:925] "Status from runtime service failed" err="rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService"
time="2022-08-13T06:00:03Z" level=fatal msg="getting status of runtime: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService"
, error: exit status 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
[root@kubemaster ~]#
[root@kubemaster ~]# rm -f  /etc/containerd/config.toml
[root@kubemaster ~]#
[root@kubemaster ~]# systemctl restart containerd
[root@kubemaster ~]#
[root@kubemaster ~]# kubeadm init
[init] Using Kubernetes version: v1.24.3
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR NumCPU]: the number of available CPUs 1 is less than the required 2
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
[root@kubemaster ~]#
[root@kubemaster ~]# kubeadm init    --ignore-preflight-errors=all
[init] Using Kubernetes version: v1.24.3
[preflight] Running pre-flight checks
        [WARNING NumCPU]: the number of available CPUs 1 is less than the required 2
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubemaster kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 172.31.45.35]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [kubemaster localhost] and IPs [172.31.45.35 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [kubemaster localhost] and IPs [172.31.45.35 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 15.003788 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node kubemaster as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node kubemaster as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: a15hlp.61zgjio72d66mriy
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.31.45.35:6443 --token a15hlp.61zgjio72d66mriy \
        --discovery-token-ca-cert-hash sha256:f40b5a10f74b94f1aaaa7d91943343e51d33c4d983633f1b7c881e95034c8a31
[root@kubemaster ~]#
[root@kubemaster ~]# crictl ps
WARN[0000] runtime connect using default endpoints: [unix:///var/run/dockershim.sock unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now deprecated, you should set the endpoint instead.
ERRO[0000] unable to determine runtime API version: rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing dial unix /var/run/dockershim.sock: connect: no such file or directory"
WARN[0000] image connect using default endpoints: [unix:///var/run/dockershim.sock unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now deprecated, you should set the endpoint instead.
ERRO[0000] unable to determine image API version: rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing dial unix /var/run/dockershim.sock: connect: no such file or directory"
CONTAINER           IMAGE               CREATED              STATE               NAME                      ATTEMPT             POD ID              POD
dcaf80975b657       2ae1ba6417cbc       About a minute ago   Running             kube-proxy                0                   0ffe90991c669       kube-proxy-dpj8h
11c4e008b7fdd       d521dd763e2e3       About a minute ago   Running             kube-apiserver            0                   af31b7b5de29d       kube-apiserver-kubemaster
5a446697f4031       3a5aa3a515f5d       About a minute ago   Running             kube-scheduler            0                   706ef0b74a220       kube-scheduler-kubemaster
d90ec000cc089       586c112956dfc       About a minute ago   Running             kube-controller-manager   0                   d8fdffb61f944       kube-controller-manager-kubemaster
2132b2bba97f6       aebe758cef4cd       About a minute ago   Running             etcd                      0                   1254affc0238f       etcd-kubemaster
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]# kubectl get  nodes
The connection to the server localhost:8080 was refused - did you specify the right host or port?
[root@kubemaster ~]#
[root@kubemaster ~]# ls -l /etc/kubernetes/admin.conf
-rw-------. 1 root root 5640 Aug 13 06:03 /etc/kubernetes/admin.conf
[root@kubemaster ~]#
[root@kubemaster ~]# kubectl get  nodes  --kubeconfig=/etc/kubernetes/admin.conf
NAME         STATUS     ROLES           AGE     VERSION
kubemaster   NotReady   control-plane   5m56s   v1.24.3
[root@kubemaster ~]#
[root@kubemaster ~]#  export KUBECONFIG=/etc/kubernetes/admin.conf
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]# kubectl get  nodes
NAME         STATUS     ROLES           AGE     VERSION
kubemaster   NotReady   control-plane   7m11s   v1.24.3
[root@kubemaster ~]#
[root@kubemaster ~]# kubectl get  componentstatus
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE                         ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health":"true","reason":""}
[root@kubemaster ~]#
[root@kubemaster ~]#
[root@kubemaster ~]# kubectl get  pod  -A
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
kube-system   coredns-6d4b75cb6d-5llr2             0/1     Pending   0          8m16s
kube-system   coredns-6d4b75cb6d-blzpt             0/1     Pending   0          8m16s
kube-system   etcd-kubemaster                      1/1     Running   0          8m21s
kube-system   kube-apiserver-kubemaster            1/1     Running   0          8m21s
kube-system   kube-controller-manager-kubemaster   1/1     Running   0          8m21s
kube-system   kube-proxy-dpj8h                     1/1     Running   0          8m16s
kube-system   kube-scheduler-kubemaster            1/1     Running   0          8m21s
[root@kubemaster ~]#





 
