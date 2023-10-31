---
#layout: posts
tags: 
  - kubernetes
  - docker
  - raspberry pi
  - cluster
  - ansible
  - k3s
title: "Setting up the Kubernetes cluster"
excerpt: "In the previous blog post I wrote about what made build a 3-node Raspberry Pi Kubernetes cluster and today I'm writing about how to set up the Pi's to work with Kubernetes, I will not cover how to install Linux on them as there is enough information on the Raspberry Pi Foundation website"
header:
  teaser: "/assets/images/k8s_2/header.jpg"
  og_image: "/assets/images/k8s_2/header.jpg"

toc: true # Table of contents
toc_sticky: True
#author_profile: false
---
<center>
<img src="{{ site.baseurl }}/assets/images/k8s_2/header.jpg" />
 <center><small><em>k3s + Raspberry Pi + Ansible</em></small></center>
 </center>
---

In the previous blog post I wrote about what made build a 3-node Raspberry Pi __Kubernetes__ cluster (1 control plane + 2 worker nodes) and today I'm writing about how to set up the Pi's to work with Kubernetes, I will not cover how to install Linux on them as there is enough information on the Raspberry Pi Foundation website, [like this guide](https://www.raspberrypi.com/documentation/computers/getting-started.html).

The software I chose to install on the Pi's is called `k3s` because it is designed to run in IoT devices as it is very lightweight. A full Kubernetes software would not run well on a Raspberry Pi 4 (or not run at all).

## Prerequisites

You are going to be running commands from your PC (__`deployment environment`__) and need to have `passwordless` SSH access to every Raspberry Pi (control plane and workers). To have this, copy the content of your `~/.ssh/id_rsa.pub` file __from your deployment environment__ to the target file `~/.ssh/authorized_keys` on all your Raspberry Pi's.

Deployment environment must have Ansible 2.4.0+ installed.

## k3s installation

We are going to take the easiest and best way of installing it, which is using `Ansible`. We will use the [oficial playbook](https://github.com/k3s-io/k3s-ansible) that k3s has uploaded to GitHub (made by [itwars](https://github.com/itwars)).

Download the [repository](https://github.com/k3s-io/k3s-ansible) to your computer and create a new folder within the `inventory` directory with a name for your cluster (in the example below is called __k3s-cluster__):

```bash
$ cp -R inventory/sample inventory/k3s-cluster
```

Edit the file `inventory/k3s-cluster/hosts.ini` to match the system information gathered above (your Raspberry Pi IP's):

```bash
[master]
192.16.35.12

[node]
192.16.35.[10:11]

[k3s_cluster:children]
master
node
```

Edit the file `inventory/my-cluster/group_vars/all.yml` to match your environment. In my case I had to change the variable `ansible_user` y `k3s_version`

```bash
$ cat all.yml
---
k3s_version: v1.23.6+k3s1
ansible_user: pi
systemd_dir: /etc/systemd/system
master_ip: "{{ hostvars[groups['master'][0]]['ansible_host'] | default(groups['master'][0]) }}"
extra_server_args: ""
extra_agent_args: ""
```

### Linux cgroups

Cgroups need to be enabled (will be enabled when running the playbook unless you have the `/boot` partition mounted as __Read-Only__). To do so, mount the partition as __Read-Write__:

Check your partition identifier with:

```bash
$ mount -v | grep "^/" | awk '{print "\nPartition identifier: " $1  "\n Mountpoint: "  $3}'
```

Example:

```bash
$ sudo mount -o remount,rw /dev/sda1 /boot
```

Then edit `/boot/cmdline.txt` and add to the end of the first line:

`cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory`

### Run playbook

At this point we are all set up to run the playbook and have Ansible do the rest:

``` bash
$ ansible-playbook site.yml -i inventory/k3s-cluster/hosts.ini
```

It will take some time to run al the tasks, but it everything goes well you should have k3s installed in all the Raspberry Pi's with special configuration applied for the __control plane__ and the __worker nodes__.

### Kubeconfig

To get access to your Kubernetes cluster just type:

```bash
$ scp pi@control_plane_IP:~/.kube/config ~/.kube/config-k3s-cluster
```

## Installing Kubectl

Lastly you need the software to communicate with the Kubernetes API, that will be `kubectl`, probably the most widely used software to communicate with k8s. The k3s and kubectl version that you install can't have a difference of +/-1 version, this means that if you have "1.23 k3s version" you can't have 1.21 or less or 1.25 or more of kubectl. This works the other way around too, so both softwares need to be on at most +1/-1 version difference.

If you needed to change the k3s version, remember to update `inventory/my-cluster/group_vars/all.yml` with the new version before running the playbook again.

To install kubectl I will follow the [official guide](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) and since I have Raspbian installed I will follow the Debian-based steps:

1.    Update the apt package index and install packages needed to use the Kubernetes apt repository:
```
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl
```
2.   Download the Google Cloud public signing key:
```
    sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```
3.    Add the Kubernetes apt repository:
```
    echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
4.    Update apt package index with the new repository and install kubectl:
```
    sudo apt-get update
    sudo apt-get install -y kubectl
```

Test that it is indeed installed checking its version:

```bash
$ kubectl version
```

To use kubectl you need to export the variable `KUBECONFIG` with the location of your `kube-config` file:

```bash
$ export KUBECONFIG=~/.kube/config-k3s-cluster
```

If everything worked you should now see your nodes:

```bash
$ kubectl get nodes
NAME         STATUS   ROLES                  AGE    VERSION
master-k3s   Ready    control-plane,master   230d   v1.23.6+k3s1
worker-02    Ready    <none>                 230d   v1.23.6+k3s1
worker-01    Ready    <none>                 230d   v1.23.6+k3s1
```

Congratulations! You are now the owner of your very own k3s cluster that is up and running and ready to have pods running on it.

## Wrapping up

In this blog post I didn't cover what can you do with your cluster once it is up and running. Perhaps install and configure an ingress controller (such as __Traefik__ or __Nginx__ `Reverse Proxy`) and run a simple webserver using a DNS resolution. Also it is a good idea to install a monitoring tool such as __Prometheus + Grafana__. I would recommend the [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) helm chart for it's easiness to install.