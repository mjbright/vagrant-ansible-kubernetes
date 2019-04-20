# ansible-kubernetes

Ansible scripts to be used with Vagrant/Terraform to create Kubernetes clusters.

## Initial version: Vagrant+Ansible+VirtualBox
The initial version is essentially what is proposed in this blog post from March 2019:
["*Kubernetes Setup Using Ansible and Vagrant*"](https://kubernetes.io/blog/2019/03/15/kubernetes-setup-using-ansible-and-vagrant/) on the Kubernetes Blog,
by Naresh L J (Infosys).

It has been slightly modified and tested using:
```
Ansible 2.7.7
Vagrant: 2.2.4
VirtualBox-5.2.24-1
```
on Fedora 28.

## Installing Kubernetes

Assuming you already have Vagrant, Ansible and VirtualBox (or other Hypervisor with suitable Vagrant driver) installed,
install Kubernetes across the cluster of VMs using the ```vagrant up``` command.

Once successfully complete check that the cluster is functioning by:

- Checking all nodes are created/running

```
vagrant status
```

which should show something like

```
Current machine states:

k8s-master                not created (virtualbox)
node-1                    not created (virtualbox)
node-2                    not created (virtualbox)
```

- logging into the master node
```
vagrant ssh k8s-master
```
```

- listing nodes of the cluster
then once logged in:
```
vagrant@k8s-master:~$ kubectl get nodes
```
```

should show something like

```
NAME         STATUS   ROLES    AGE     VERSION
k8s-master   Ready    master   8h      v1.14.1
node-1       Ready    <none>   6h6m    v1.14.1
node-2       Ready    <none>   5h58m   v1.14.1
```


## To Add to this version

- Recuperate kubeconfig info locally
- Test kubecongfig locally
- Create some pods, services and test access

## Future versions

The intention is to keep this repo updated with best practices and ideas from other implementations, for example:

- https://github.com/kairen/kubeadm-ansible
- https://github.com/alisonbuss/cluster-kubernetes-ansible-vagrant
- http://michele.sciabarra.com/2018/02/12/devops/Kubernetes-with-KubeAdm-Ansible-Vagrant/
  -  https://github.com/sciabarracom/Mosaico3
- https://github.com/jeremievallee/kubernetes-vagrant-ansible

and using other tools:
- rpm(/yum/dnf)-based distributions
- rancher/k3s
- etcdadm, nodeadm, ...


