---
#layout: posts
tags: 
  - tutorial
  - guide
  - vagrant
  - ansible
  - wsl
title: "Creating OS agnostic Ansible Playbooks"
excerpt: "How to provision a Vagrant machine with Ansible code in your test environment in any OS (Windows compatible), hassle free by just installing one plugin"
header:
  teaser: "/assets/images/ansible-vagrant.jpg"
  og_image: "/assets/images/ansible-vagrant.jpg"
toc: true # Table of contents
toc_sticky: True
#author_profile: false
---

<center>

<img src="{{ site.baseurl }}/assets/images/ansible-vagrant.jpg" />
 <center><small><em>Vagrant + Ansible</em></small></center>
 </center>

----

In [a previous blog post](https://slavid.github.io/2021/11/28/running-vagrant-ansible-windows-through-wsl2/) I wrote how to run Ansible on Windows 10. It was a rather complex process to set up, it required WSL installed and running it would fail with random errors fairly often, mostly related to how the Virtual Machine that runs the Linux Kernel for WSL had to communicate with Windows 10 and to the other Virtualbox machine deployed with Vagrant. This new method doesn't require WSL (although I strongly recommend to have WSL installed anyway) and does not fail every now and then like the previous method.

We will be using the plugin `vagrant-guest_ansible`.

## Installation

This should be run on Windows, since both Linux and macOS can run Ansible natively:

```bash
$ vagrant plugin install vagrant-guest_ansible
```

## Use

Inside a `Vagrantfile` add a variable `provisioner` which depending on the platform you are using will use the `guest_ansible` provisioner or the default ansible provisioner. 

```vagrantfile
Vagrant.configure("2") do |config|

  config.vm.box = "generic/ubuntu2204"
  config.vm.synced_folder ".", "/vagrant", disabled: false

  provisioner = Vagrant::Util::Platform.windows? ? :guest_ansible : :ansible
  # Optional, output which provisioner will be used (for debugging purposes): puts "Provisioner that will be used: #{provisioner}"
  config.vm.provision provisioner do |ansible|
      ansible.playbook = "playbook.yml"
  end
end
```

What we can see is that if the platform is `windows` then it will use `guest_ansible`, that means ansible will be installed and run in the guest machine instead of the host. If not using windows (Linux or `darwin`=macOS) then it will use ansible in the host machine.

Adding that line to every Vagrantfile will not affect any of your running Vagrantfiles if you are already working on Linux or macOS, but will make the work as well in Windows. So if working in a team with developers using multiple OSes that will make the Vagrantfiles OS-agnostic and easy the workflow.

## Credits

- https://github.com/vovimayhem/vagrant-guest_ansible