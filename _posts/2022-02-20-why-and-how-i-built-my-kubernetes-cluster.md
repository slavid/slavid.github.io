---
#layout: posts
tags: 
  - kubernetes
  - docker
  - raspberry pi
title: "Why and how I built a Raspberry Pi cluster"
excerpt: "My thoughts on why building a 3-node kubernetes cluster with Raspberry Pi was a good idea for me to learn."
header:
  teaser: "/assets/images/k8s/header.png"
  og_image: "/assets/images/k8s/header.png"

toc: true # Table of contents
toc_sticky: True
#author_profile: false
---
<center>
<img src="{{ site.baseurl }}/assets/images/k8s/header.png" />
 <center><small><em>A cluster of Raspberry Pi</em></small></center>
 </center>
---

It was **March 2013** that [**Docker debuted to the public**](https://www.youtube.com/watch?v=wW9CAH9nSLs) in Santa Clara at PyCon 2013 and blown away the whole world of IT. They showed a technology capable of:
- Creating a LXC Container (before switching to their own container technology *libcontainer*).
- Allocating a read-write filesystem to the container.
- Creating a network interface to connect the container to the default network.
- Setting up NAT.
- Executing a process.
- Capture the output of that process in the terminal.

It didn't take long until __Docker became the industry standard in containers__ because of their easiness to use and because how much it could help developers, infrastructure deployment, self hosters and such. Like many others I started using it as soon as I heard about it, and again like many others, my first Docker container was [__Pi-Hole__](https://pi-hole.net/).

At some point I was using an `ingress controller` working with my local `DNS server` to access local services with URL instead of "IP:Port", using `secrets`, using `proxy-sockets`, `multiple networks`, etc... I felt like the king of this world; but then... I heard about __Kubernetes__:

>That\
Changed\
Everything

## Motivation

My first impression with *Kubertenes* (in short __k8s__ as abbreviation of Kubernetes) was that if felt very complex unlike Docker that I could be playing with it in just a matter of days, k8s felt like it would need more time to fully unleash its true potential. And *boy, was I right*.

I spent days reading as much as I could and watching crash courses about k8s and still didn't feel comfortable enough to try myself. But have to leave fears behind and give it a try so I decided to go for the hardcore way: __build a bare metal kubernetes cluster__.

## Raspberry Pi cluster

<center>
<img src="{{ site.baseurl }}/assets/images/k8s/rpi_01.jpg" />
 <center><small><em>3 unpacked Raspberry Pi's</em></small></center>
 </center>

I chose __Raspberry Pi__ as the underlying hardware because I had already built a cluster of Raspberry Pi as part of [__my final project in the University__](https://repositorio.unican.es/xmlui/bitstream/handle/10902/9383/Lavid%20Ortiz%20Salvador.pdf?sequence=1). But buying a Raspberry Pi these days is not an easy task as there is a huge demand and a low production for the public market (most of the Raspberry Pi produced are sold to private corporations that the `Raspberry Pi Foundation` has deals with). This is partialy derived from the semiconductor crisis where semiconductors are in high demand, and in short supply.

I wanted 3 `Raspberry Pi 4 Model B 2 GB` to build a 3-node cluster, saved some money and waited and waited until one day there was supply. Together with the 3 Raspberry Pi's I bought a PoE Switch (to power the Raspberry Pi with just the ethernet cables), 3 PoE Hats, Ethernet cables, 3 USB Flash Drives and the Rack Mount and this is what it cost:

--------------------

1 x Switch Hored AI106G - Smart Switch Gigabit PoE = 49.99€\
1 x Ethernet Cables = 9.90€\
3 x Raspberry Pi Model 4B 2GB = 3x62 = 186€\
3 x PoE Hat = 3x21,92 = 66,32€\
1 x Rpi Rack Mount = 19,99€\
3 x USB storage pendrives = 20,99€

--------------------

**Total** = 186 + 66,32 + 49,99 + 9,90 + 19,99 + 20,99 = **353,19€**

<center>
<img src="{{ site.baseurl }}/assets/images/k8s/rpi_02.jpg" />
 <center><small><em>Mounted Raspberry Pi Rack</em></small></center>
 </center>
 
 I wanted to have a cluster of Raspberry Pi's that could power from the ethernet cables to reduce the cable clutter just like [Jeff Geerling](https://github.com/geerlingguy/) has on his [Pi Dramble project](https://www.pidramble.com/), and well the expectations were high.
 
 <center>
<img src="{{ site.baseurl }}/assets/images/k8s/expectation-reality-2.png" />
 <center><small><em>Expectation vs Reality (graphic design is my passion)</em></small></center>
 </center>
 
>But hey! Performance is what really matters and at the end of the day you are going to have the cluster placed somewhere it is not very visible so looks shouldn't be that very important after all.

## What to do once is assembled

In the next blog post I will write about the first steps to do in order to have a Kubernetes cluster up and running.