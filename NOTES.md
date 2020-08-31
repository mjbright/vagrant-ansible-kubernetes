
# Based on blog post from March 2019 here:
#    https://kubernetes.io/blog/2019/03/15/kubernetes-setup-using-ansible-and-vagrant/

# For info on using Vagrant & Ansible together:
#    https://docs.ansible.com/ansible/latest/scenario_guides/guide_vagrant.html


# WARNING: Due to Virtualbox need to specify node_ip option on kubelet

See: https://stackoverflow.com/questions/59629319/unable-to-upgrade-connection-pod-does-not-exist

Original ansible YAML from above post had:
```yaml
 - name: Configure node ip
    lineinfile:
      path: /etc/default/kubelet
      line: KUBELET_EXTRA_ARGS=--node-ip={{ node_ip }}
```

New version adapted to kubeadm (and later Kubernetes release?) install:
```yaml
   - name: Configure node ip
     lineinfile:
       #path: /etc/default/kubelet
       path: /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
       line: KUBELET_EXTRA_ARGS=--node-ip={{ node_ip }}
       insertafter: '\[Service\]'
```


# WARNING: Due to Virtualbox bug nodes have same MAC(and so IP) on enp:wq

    link/ether 02:8a:e8:1b:bb:fd brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3

NOTE in INTERNAL-IP column, all nodes have the same IP:
    $ vagrant ssh master -- kubectl get no -o wide
    NAME      STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
    master    Ready    master   8m46s   v1.19.0   10.0.2.15     <none>        Ubuntu 18.04.5 LTS   4.15.0-112-generic   docker://19.3.12
    worker1   Ready    <none>   6m47s   v1.19.0   10.0.2.15     <none>        Ubuntu 18.04.5 LTS   4.15.0-112-generic   docker://19.3.12
    worker2   Ready    <none>   4m10s   v1.19.0   10.0.2.15     <none>        Ubuntu 18.04.5 LTS   4.15.0-112-generic   docker://19.3.12
    worker3   Ready    <none>   2m19s   v1.19.0   10.0.2.15     <none>        Ubuntu 18.04.5 LTS   4.15.0-112-generic   docker://19.3.12

BUT NOT A PROBLEM A PRIORI: see NODE column shows node names correctly    
    $ vagrant ssh master -- kubectl get pods -o wide
    NAME                         READY   STATUS    RESTARTS   AGE    IP                NODE      NOMINATED NODE   READINESS GATES
    ckad-demo-74d675dbc4-5xwz7   1/1     Running   0          2m8s   192.168.189.65    worker2   <none>           <none>
    ckad-demo-74d675dbc4-dg2wq   1/1     Running   0          66s    192.168.189.67    worker2   <none>           <none>
    ckad-demo-74d675dbc4-hd6tj   1/1     Running   0          66s    192.168.235.130   worker1   <none>           <none>
    ckad-demo-74d675dbc4-jqrgm   1/1     Running   0          66s    192.168.235.132   worker1   <none>           <none>
    ckad-demo-74d675dbc4-m7kj5   1/1     Running   0          66s    192.168.189.66    worker2   <none>           <none>
    ckad-demo-74d675dbc4-r7t95   1/1     Running   0          66s    192.168.182.2     worker3   <none>           <none>
    ckad-demo-74d675dbc4-rxmdj   1/1     Running   0          66s    192.168.235.129   worker1   <none>           <none>
    ckad-demo-74d675dbc4-vt9mn   1/1     Running   0          66s    192.168.235.131   worker1   <none>           <none>
    ckad-demo-74d675dbc4-vzjl7   1/1     Running   0          66s    192.168.182.1     worker3   <none>           <none>
    ckad-demo-74d675dbc4-w7572   1/1     Running   0          66s    192.168.182.3     worker3   <none>           <none>

# TODO: Once working
- DONE: archive reference version (Vagrantfile and kubernetes-setup/.yml)
- DONE: modify master name to 'master', node-N to workerN
- DONE: More memory => modify v.memory line, then 'vagrant reload' (reboots)
- Use other base image: ubuntu1804 (2004?)
- Install linux tools: htop
- install from kubeadm.yaml => set ephemeral feature-gate
- Increase memory per node
- Archive code
- Install utilities: krew, kubebox, k9s, helm, arkade, k8scenario
- Optional: install complete application stacks + ckad-demo
- Do a k3s version?
- Do a rke version?
- make resilient to multiple kubeadm init (or prevent based on ... ?)
- make resilient to kubeadm join timeouts (why so slow?)
  - increase timeout?
  - retry?

- later:
  - multiple master nodes
  - external etcd nodes


### ip

$ vagrant ssh master -- ip a | grep enp0s3 -A 2
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:8a:e8:1b:bb:fd brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 85901sec preferred_lft 85901sec
    inet6 fe80::8a:e8ff:fe1b:bbfd/64 scope link

$ vagrant ssh worker1 -- ip a | grep enp0s3 -A 2
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:8a:e8:1b:bb:fd brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 86073sec preferred_lft 86073sec
    inet6 fe80::8a:e8ff:fe1b:bbfd/64 scope link

$ vagrant ssh worker2 -- ip a | grep enp0s3 -A 2
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:8a:e8:1b:bb:fd brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 86171sec preferred_lft 86171sec
    inet6 fe80::8a:e8ff:fe1b:bbfd/64 scope link

$ vagrant ssh worker3 -- ip a | grep enp0s3 -A 2
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:8a:e8:1b:bb:fd brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 86319sec preferred_lft 86319sec
    inet6 fe80::8a:e8ff:fe1b:bbfd/64 scope link


