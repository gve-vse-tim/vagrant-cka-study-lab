# Multinode Kubernetes Control Plane Creation

My instructions on how to build a multi-node control plane based Kubernetes cluster.
I followed the Kubernetes High Availability documentation for the most part:

- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/

However, as a requirement for high-availability, you must provide a virtual IP service
for the Kubernetes API service to leverage.  I chose kube-vip because I didn't want to
run external services (keepalived and haproxy) in this environment.

## VIP via Kube-VIP

The Kubernetes GitHub repo for kubeadm provided these instructions:

- https://github.com/kubernetes/kubeadm/blob/master/docs/ha-considerations.md#options-for-software-load-balancing

But unfortunately they were stale, based on the kube-vip.io website:

- https://kube-vip.io/hybrid/
- https://github.com/kube-vip/kube-vip/issues/186

Created the correct kube-vip.yaml configuration file:

```bash
sudo docker run --network host --rm plndr/kube-vip:0.3.4 manifest pod \
  --interface enp0s8 --arp \
  --vip 192.168.56.10 \
  --controlplane --leaderElection 
```

Of course, I stored that in /etc/kubernetes/manifests/kube-vip.yaml of the deployed
control nodes.  Vagrant will copy that file from the config subdirectory inside
the VM.

## Vagrant

You need to spin up the multi-node environment in Virtualbox via Vagrant and the
provided [Vagrantfile](./Vagrantfile):

```bash
vagrant up
```

## KubeADM

Of course, I'm going to use **kubeadm** to build the Kubernetes cluster. I'm practical.
The explanation for the various CLI arguments provided:

- I'm ignoring the pre-flight error on number of CPUs because I'm building 6 VMs on my  laptop and didn't want to allocate 9 vCPUs.
- The pod CIDR chosen matches (Flannel's default CIDR) is used to override Weave's default CIDR, which conflicts with my VirtualBox network.
- The control plane endpoint is the virtual IP (VIP) for the Kubernetes API server
- The API Server advertise address is the actual IP for the control node API IP. kube-vip knows this IP and will loadbalance traffic from the VIP to each control node's API IP.

### Control Plane Node 1

```bash
# control1
sudo kubeadm init --kubernetes-version=1.20.10 \
   --upload-certs --ignore-preflight-errors=NumCPU,Mem,SystemVerification \
   --pod-network-cidr=10.244.0.0/16 \
   --control-plane-endpoint=192.168.56.10 \
   --apiserver-advertise-address=192.168.56.11

## Save the commands in the output for the control and worker node joins

# Setup KUBECONFIG for kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Deploy Weave CNI
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

```

The commands from the **kubeadm init** output have the form like:

```bash
# For other control plane nodes
  kubeadm join 192.168.56.10:6443 --token yt0kw4.fedb8c5ps8joiev8 \
    --discovery-token-ca-cert-hash sha256:6d5f3f5fa17d1624b7a0ea7e0a5dad4bf7abeb16fc48cf63e34d4850ec9569e4 \
    --control-plane --certificate-key e5aa68bd0287d437d76820e980c7cb9aaa8597100bb65debecaa85e7ce613d9b

# For worker nodes
kubeadm join 192.168.56.10:6443 --token yt0kw4.fedb8c5ps8joiev8 \
    --discovery-token-ca-cert-hash sha256:6d5f3f5fa17d1624b7a0ea7e0a5dad4bf7abeb16fc48cf63e34d4850ec9569e4 
```

### Control Plane Node 2

The **kubeadm join** checked for whether the Kubernetes manifest directory was
empty.  Since Vagrant copies in the kube-vip.yaml manifest, it clearly is not
empty any longer.  So the primary difference in this command is the addition
of that pre-flight error to the ignored checks list.

```bash
## control2
sudo kubeadm join 192.168.56.10:6443 --token yt0kw4.fedb8c5ps8joiev8 \
    --discovery-token-ca-cert-hash sha256:6d5f3f5fa17d1624b7a0ea7e0a5dad4bf7abeb16fc48cf63e34d4850ec9569e4 \
    --control-plane --certificate-key e5aa68bd0287d437d76820e980c7cb9aaa8597100bb65debecaa85e7ce613d9b \
    --ignore-preflight-errors=NumCPU,SystemVerification,Mem,DirAvailable--etc-kubernetes-manifests \
    --apiserver-advertise-address=192.168.56.12

# Setup KUBECONFIG for kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

### Control Plane Node 3

```bash
## control3
sudo kubeadm join 192.168.56.10:6443 --token yt0kw4.fedb8c5ps8joiev8 \
    --discovery-token-ca-cert-hash sha256:6d5f3f5fa17d1624b7a0ea7e0a5dad4bf7abeb16fc48cf63e34d4850ec9569e4 \
    --control-plane --certificate-key e5aa68bd0287d437d76820e980c7cb9aaa8597100bb65debecaa85e7ce613d9b \
    --ignore-preflight-errors=NumCPU,SystemVerification,Mem,DirAvailable--etc-kubernetes-manifests \
    --apiserver-advertise-address=192.168.56.13

# Setup KUBECONFIG for kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Worker Nodes

For each and every worker node:

```bash
sudo kubeadm join 192.168.56.10:6443 --token yt0kw4.fedb8c5ps8joiev8 \
    --discovery-token-ca-cert-hash sha256:6d5f3f5fa17d1624b7a0ea7e0a5dad4bf7abeb16fc48cf63e34d4850ec9569e4 \
    --ignore-preflight-errors=SystemVerification
```
