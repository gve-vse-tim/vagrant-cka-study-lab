# Study Notes for Upgrading Kubernetes in Cluster

General workflow as based on
[Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
at kubernetes.io:

- Upgrade kubeadm
- Run upgrade plan and apply
- Drain control plane node
- Upgrade kubectl and kubelet
- Restart kubelet
- Uncordon control plane node
- Repeat for all control plane nodes
- Repeat for work nodes (plan/apply is omitted for simple upgrade)

## Example for single control plane cluster

These were commands upgrading K8s 1.19.10 to 1.20.7:

```bash
# control1
sudo apt-get install kubeadm=1.20.7-00
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.20.7
kubectl drain control --ignore-daemonsets=true
sudo apt-get install kubectl=1.20.7-00 kubelet=1.20.7-00
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl uncordon control

# Node1
sudo apt-get install kubeadm=1.20.7-00
sudo kubeadm upgrade node
kubectl drain node1 --ignore-daemonsets=true
sudo apt-get install kubectl=1.20.7-00 kubelet=1.20.7-00
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl uncordon node1

# Node2
sudo apt-get install kubeadm=1.20.7-00
sudo kubeadm upgrade node
kubectl drain node2 --ignore-daemonsets=true
sudo apt-get install kubectl=1.20.7-00 kubelet=1.20.7-00
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl uncordon node2
```
