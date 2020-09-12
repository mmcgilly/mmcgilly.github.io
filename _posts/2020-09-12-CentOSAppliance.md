---
layout: post
title: CentOS Appliance
tags: VMware CentOS Packer
---
I've been working on creating a vSphere appliance for CentOS for easy deployments. I've applied the same technique used in [Photon OS as an SRM Test VM](https://mmcgilly.github.io/PhotonSRM/). It's the solid process that William Lam devised and is used to build [VEBA, the VMware Event Broker Appliance](https://github.com/vmware-samples/vcenter-event-broker-appliance). 

Compared to Photon OS it's a pretty beefy OVA at 820MB.  There might be opportunities I've missed to slim it down even further, a task for another day. 

Similar to the Photon OS appliance the concept is that you can specify all the key parameters at deploy time and you don't need to manually login to configure users, network interfaces or disks. 

I have added to ability to inject an SSH key to the root user's authorized_keys. For me this then allows further configuration with Ansible. 

Full details on how to build the appliance are included in the [GitHub repository](https://github.com/mmcgilly/centos-appliance). 
