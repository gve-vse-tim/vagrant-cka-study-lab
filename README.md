# Kubernetes Cluster Vagrant Files

This repository has Vagrant content, notes, and instructions for building
small, lab ready, Kubernetes clusters on your local laptop.  I have used
these resources for studying for the Certified Kubernetes Administrator
exam.

There are two Vagrantfiles: a [single node control plane](single/Vagrantfile)
and a [multi-node control plane](multi/Vagrantfile)

These files build the virtual machines in VirtualBox on your laptop and
set up the pre-requisites for leveraging kubeadm to install/configure
your cluster.
