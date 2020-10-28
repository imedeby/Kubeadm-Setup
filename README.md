# Kubeadm-Setup On CentOS 8

## On All Nodes

### Environment network

```
        Master   192.168.60.100
        Node-1   192.168.60.101
        Node-2   192.168.60.102
```
### Docker Install

```
        yum-config-manager -- add-repo https://download.docker.com/linux/centos/docker-ce.repo
        yum install -y docker-ce
        systemctl enable --now docker
        dnf install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
        systemctl enable --now cri-o
```
### Firewall Configuration

```
        firewall-cmd --permanent --add-port=6443/tcp
        firewall-cmd --permanent --add-port=2379-2380/tcp
        firewall-cmd --permanent --add-port=10250/tcp
        firewall-cmd --permanent --add-port=10251/tcp
        firewall-cmd --permanent --add-port=10252/tcp
        firewall-cmd --permanent --add-port=10255/tcp
        firewall-cmd --reload
        modprobe br_netfilter
        cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
        net.bridge.bridge-nf-call-ip6tables = 1
        net.bridge.bridge-nf-call-iptables = 1
        EOF
        sysctl --system
 ```
 
 ### System  Configuration
 
 ```
        sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
        swapoff -a
        remove swap from fstab (vim /etc/fstab)
 ```
 ### Kubernetes install
 
 ```
        cat <<EOF > /etc/yum.repos.d/kubernetes.repo
        [kubernetes]
        name=Kubernetes
        baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
        enabled=1
        gpgcheck=1
        repo_gpgcheck=1
        gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
        EOF
        yum install -y kubelet kubeadm kubectl
        systemctl enable --now kubelet
 ```
## On Master Node
### Initialise our cluster

```
        kubeadm init --pod-network-cidr=10.244.0.0/16
        mkdir -p $HOME/.kube
        cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        chown $(id -u):$(id -g) $HOME/.kube/config
```   

### Setup Pods Network before joning the worker node 

```
        kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
## On Worker Node
### Join your Master cluster 

```
        kubeadm join 192.168.60.100:6443 -- token yn8psk.z2h7sqg2rtw5mquz — discovery-token-ca-cert-hash sha256:2c4fc54c3036ed5722f5458c9eb03287d6264b3c3f9fb92e5dbe1c9feb2dd045
```
        

