# Single Node Kubernetes Control Plane Creation

My instructions on how to build a single-node control plane based Kubernetes cluster.
This repo focuses on using **kubeadm** because I am not the guru that is the one
and only [Kelsey Hightower](https://github.com/kelseyhightower/kubernetes-the-hard-way)

- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

## Vagrant

You need to spin up the multi-node environment in Virtualbox via Vagrant and the
provided [Vagrantfile](./Vagrantfile):

```bash
vagrant up
```

## KubeADM

Of course, I'm going to use **kubeadm** to build the Kubernetes cluster. I'm practical.
The explanation for the various CLI arguments provided:

- I'm ignoring the pre-flight error on number of CPUs because I'm building 6 VMs on my laptop and didn't want to allocate 9 vCPUs.
- The pod CIDR chosen matches (Flannel's default CIDR) is used to override Weave's default CIDR, which conflicts with my VirtualBox network.
- The control plane endpoint is the virtual IP (VIP) for the Kubernetes API server
- The API Server advertise address is the actual IP for the control node API IP. kube-vip knows this IP and will loadbalance traffic from the VIP to each control node's API IP.

### Control Plane Node 1

```bash
# control1
sudo kubeadm init --kubernetes-version=1.20.10 \
   --ignore-preflight-errors=NumCPU,Mem,SystemVerification \
   --pod-network-cidr=10.244.0.0/16 \
   --control-plane-endpoint=192.168.56.11 \
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
# For worker nodes
kubeadm join 192.168.56.11:6443 --token olv9lm.fn8bfwe3ro2rry8x \
    --discovery-token-ca-cert-hash sha256:3332e95a8283e09ddd5998c01ac8c0085e9cb090f0f160e4cd5089020e9e2a4d
```

### Worker Nodes

For each and every worker node:

```bash
sudo kubeadm join 192.168.56.11:6443 --token olv9lm.fn8bfwe3ro2rry8x \
    --discovery-token-ca-cert-hash sha256:3332e95a8283e09ddd5998c01ac8c0085e9cb090f0f160e4cd5089020e9e2a4d \
    --ignore-preflight-errors=SystemVerification
```
