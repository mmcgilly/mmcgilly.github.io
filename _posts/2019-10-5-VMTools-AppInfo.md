---
layout: post
title: VMware Tools 11 - New AppInfo component
---

The latest (11.0) release of VMware Tools is out and one interesting feature that caught my attention was a new component call AppInfo. This component collects information of running processes inside the guest OS. It then makes this information available through normal VMTools procedure to the vSphere Admin. That means we can access guest OS data out-of-band and without needing guest OS credentials. Pretty cool!

I wasn't able to find any more detail on exactly what data AppInfo collected either in the release notes or elsewhere on the web.
I decided to find out for myself. Spinning up a Windows Server 2016 VM, I installed VMware Tools 11.0.

![VMInfo]({{ site.baseurl }}/images/VMTools-appinfo-vminfo.png)


### Inside the Guest OS
The [Configuring appInfo ](https://docs.vmware.com/en/VMware-Tools/11.0.0/com.vmware.vsphere.vmwaretools.doc/GUID-3A8089F6-CAF6-43B9-BD9D-B1081F8D64E2.html)  section of the VMWare Tools Product Documentation gives a good idea of what we can do from within the guest OS. There isn’t all that much to it.

We can retrieve the data

![Retrieve appinfo]({{ site.baseurl }}/images/VMTools-appinfo-rpctool.png)

Change the polling interval. Here I have reduced it from the default 30 minutes to 30 seconds while I explore the feature.

![Changing poll interval]({{ site.baseurl }}/images/VMTools-appinfo-poll-interval.png)

And for Guest OS admin to stop the vSphere Admin from knowing what processes they are running, it can be disabled.

![Changing poll interval]({{ site.baseurl }}/images/VMTools-appinfo-disabled.png)

### Outside the Guest OS
It's a lot more interesting to be able to access AppInfo from outside the Guest OS.
Using PowerCLI 

![Using PowerCLI]({{ site.baseurl }}/images/VMTools-appinfo-pcli.png)

Interestingly the data is in the JSON format but it's easy to convert to something PowerShell can consume.

![Convert from JSON]({{ site.baseurl }}/images/VMTools-appinfo-pcli-json.png)

Then we can check if a particular program is running.

![Convert from JSON]({{ site.baseurl }}/images/VMTools-appinfo-pcli-iexplore.png)

What's this !? Internet Explorer is running on our Windows Server, tut tut