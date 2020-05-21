---
layout: post
title: VMware Photon OS as an SRM Test VM
tags: VMware Photon SRM Packer Ubuntu 
---

I've spent a lot of time recently working on a lab install of Site Recovery Manager and vSphere Replication. As well as the installation and configuration of SRM and vSphere Replication I wanted to be able to replicate, protect and recover VMs. In my homelab I have a two site setup running nested ESXi on a single physical ESXi host.

Having a lightweight OS to run on the nested ESXi helps save the limited physical resources. There are quite a [few options](https://cloudarchitectblog.wordpress.com/2015/11/11/yvm-download-page/) for lightweight VMs and I've used some of these in the past. This time I wanted something that I knew was up to date, had VMTools and I could use to install other software and tools. One good candidate is Ubuntu. Canonical provides an OVA from https://cloud-images.ubuntu.com/ which makes it very easy to deploy.  Luc Dekens has a [great guide](https://www.lucd.info/2019/12/06/cloud-init-part-1-the-basics/) on using cloud-init to deploy the Ubuntu OVA. Luc also has a good guide to [deploying Photon](https://www.lucd.info/2018/08/14/deploy-photon-2-0-part-1/).  Since it can be installed with a smaller footprint I went with [VMware Photon](https://vmware.github.io/photon/).

I don't want to spend a lot of time configuring the OS instances and wanted a one shot deploy from OVA. It's just neater than creating a VM Template and deploying from it.
Deploying an OVA and configuring IP addresses, root passwords etc all at the same time was a key feature I wanted. Unfortunately the official Photon OVA doesn’t let you do this.

I used William Lam's [reference packer build for PhotonOS](https://www.virtuallyghetto.com/2019/11/packer-reference-for-building-photonos-virtual-appliance-using-ovf-properties.html) as a starting point for customisation.
I'll explain the changes I made with the diffs as generated from [my fork on GitHub](https://github.com/mmcgilly/photonos-appliance/tree/photon-srm).

**photon-version.json**

![photon-version.json]({{ site.baseurl }}/images/PhotonSRM/photon-version.json_1.png)

I made a change to use the most recent Photon 3.0 Rev 2 Update 1 ISO.
From the download page the only checksums available are sha1 and md5 so I change the iso_checksum_type to use sha1

**photon.json**

![photon.json]({{ site.baseurl }}/images/PhotonSRM/photon.json_1.png)

One of the main requirements was for SRM to be able to apply IP customiztion. Leaving the guest_os_type as Other confuses SRM and the IP customization is skipped. Although Photon OS doesn't appear on the [Guest OS Customization Support Matrix](http://partnerweb.vmware.com/programs/guestOS/guest-os-customization-matrix.pdf), changing the guest_os_type to vmware-photon-64 does work.
Another requirement was minimal use of resources so I reduced the disk_size from 12GB to 8GB.

**photon-settings.sh**

![photon-settings.json1]({{ site.baseurl }}/images/PhotonSRM/photon-settings.sh_1.png)

This is where I start to diverge from the point of Photon. I remove Docker which saves roughly 250MB when the OVA is deployed. The disk space used in / is only 400MB so removing docker was a big drop proportionally.

![photon-settings.json2]({{ site.baseurl }}/images/PhotonSRM/photon-settings.sh_2.png)

The packages for less and curl are already installed so I don't try and installed them separately.
For guest OS customization Perl is required so I install it here.

![photon-settings.json3]({{ site.baseurl }}/images/PhotonSRM/photon-settings.sh_3.png)

I use lab.local as the domain name in my homelab. Using .local might not be the best idea but it's a lot of work for me to change it now. This change to DNS lookups allows proper resolution of my lab.local domain.

**setup.sh**

![setup.sh1]({{ site.baseurl }}/images/PhotonSRM/setup.sh_1.png)

I think this is a typo, the Domain entry should be Domains. It controls the DNS suffix search list.
The next 2 lines remove IPv6 but it takes an additional reboot after deploying the OVA to take effect.


![setup.sh2]({{ site.baseurl }}/images/PhotonSRM/setup.sh_2.png)

Finally because this is a test system I enable ping by default.

These various tweaks results in 153MB OVA file, which I think is hard to beat. For comparison the Ubuntu 20.04 OVA is 493MB

To allow the packer build with vmware-iso I used a community VIB - https://github.com/umich-vci/packer-vib. It enables SSH and the GuestIPHack but it was the firewall change to allow VNC that was the big change.
