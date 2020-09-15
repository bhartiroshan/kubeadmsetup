# Bootstrap your Kubernetes Cluster with kubeadm - Amazon EC2 

# Instructions 

## Step 1 on Master node:

### Spin up Master/Worker nodes:
- For example 1 Master 2 Workers(below are the specs that I used)/
- Kube Master 01(2 core 4 GB Ram 40 GB EBS)
- Kube Worker 01(2 core 16 GB Ram 40 GB EBS)
- Kube Worker 02(2 core 16 GB Ram 40 GB EBS)

### Open ports between these these instances.

- [See this doc for required ports](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports)

### Install below packages

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
```

- yum install -y kubelet kubeadm kubectl docker 
- systemctl enable kubelet && systemctl start kubelet && systemctl start docker && systemctl enable docker

## Disable SElinux
```
setenforce 0
modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

### Initialize Kube control plane using kubeadm

- kubeadm init --pod-cidr-network=10.240.0.0/16
- This command would leave below results, see example.
```
kubeadm join 172.31.14.217:6443 --token kml3du.jgbla332mjx9hd1s \
     --discovery-token-ca-cert-hash sha256:shacode 
```

## Step 2 on Worker nodes:
- Follow all steps mentioned in Master nodes except for `kubeadm init` part. 
- use the `kubeadm join` command from master node to join the cluster. 

## Step 3:

### Installing CNIs

- At this point your cluste is setup but the coredns pods may still be in pending state. 
- You need to install pod network. 
- You can install weavenet or other CNIs. 
-  kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

## Step 4:

### Installing Storage class


