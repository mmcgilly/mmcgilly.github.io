---
layout: post
title: VMC REST API - PowerSplitting
tags: PowerShell REST API VMC BGP
---

I've been working a lot recently on automating VMware Cloud on AWS. For Direct Connect and Route Based VPNs we needed to set the BGP ASN.

In the VMC console the Route Based VPN ASN is set under Networking & Security > VPN > Route Based > Edit Local ASN

![Route Based VPN BGP ASN]({{ site.baseurl }}/images/PowerShellSplit/RouteBasedVPN_BGPASN.png)

For Direct Connect the setting is under Networking & Security > Direct Connect > BGP Local ASN

![Route Based VPN BGP ASN]({{ site.baseurl }}/images/PowerShellSplit/DXASN.png)

In code these can be changed but they are contained in different APIs.

For Direct Connect the setting is part of the VGW and can be changed with the NSX VMC AWS Integration API.

For Route Based VPNs the endpoint is the NSX T0 Edge Router and can be changed with the NSX VMC Policy API.

We can access either of these APIs through the NSX-T Proxy URL.
This is what the proxy URL looks like. (I've anonymised the IP, Org Id and SDDC Id)

https://nsx-4-344-237-32.rp.vmwarevmc.com/vmc/reverse-proxy/api/orgs/3aada771-8cc3-4440-b715-2cd71e40e873/sddcs/6b72a771-701e-4ef4-9a35-35b38b3157a2/sks-nsxt-manager

It can be used directly for the NSX VMC Policy API but for the NSX VMC AWS Integration API I found I need to strip the /sks-nsxt-manager route and replace it with /cloud-service/api/v1/infra/direct-connect/bgp.

PowerShell is our language of choice for this automation and on my development machine with PowerShell 6 I could use the .split method which gets me the URL I want plus a blank string.

![PowerShell 6 split]({{ site.baseurl }}/images/PowerShellSplit/PS6Split.png)

But when this code is ran in the CI/CD pipeline it runs in PowerShell 5 and that was a problem.

![PowerShell 5 split]({{ site.baseurl }}/images/PowerShellSplit/PS5Split.png)

There is a bit of a difference between a REST API endpoint of

https://nsx-4-344-237-32.rp.vmwarevmc.com/vmc/reverse-proxy/api/orgs/3aada771-8cc3-4440-b715-2cd71e40e873/sddcs/6b72a771-701e-4ef4-9a35-35b38b3157a2 

and 

h

This caused an awkward delay in getting some production SDDCs built.

I was baffled for a while but it was easy fix once I realized what was going.
Rather than splitting on the entire string the .split method splits on each character. So it splits on **/** and on **s** and **k** on **s** etc.

Changing from .split to -split made the behaviour consistent across PowerShell 5 and 6.

![PowerShell 6 -split]({{ site.baseurl }}/images/PowerShellSplit/PS6-Split.png)
![PowerShell 5 -split]({{ site.baseurl }}/images/PowerShellSplit/PS5-Split.png)

This explains how .split works better - [https://devblogs.microsoft.com/scripting/powertip-know-the-difference-between-the-split-method-and-split/](https://devblogs.microsoft.com/scripting/powertip-know-the-difference-between-the-split-method-and-split/)
