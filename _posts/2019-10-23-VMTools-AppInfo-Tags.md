---
layout: post
title: VMware Tools 11 - Tagging with AppInfo
---

This post follws on from my blog about the new AppInfo component in VMware Tools 11.0

[VMware Tools 11 - New AppInfo component]({% post-url 2019-10-5-VMTools-AppInfo %})

Now that we can see process info from inside a guest OS the question becomes - what can do with it?

I'll lay out here a technique to read the appInfo and if a process matches a known process then add a tag.
For example if there is a mssqlserver.exe process then we know the VM is running MS SQL Server. Once we surface this information as a tag then it is a lot easiest to take action.

### Set-ApplicationTag
I wrote a small function to automate the reading of appInfo and assigning tags.

{% highlight powershell %}
Function Set-ApplicationTag {
    [CmdletBinding()]
    Param (
        [Parameter(Mandatory = $true)]
        [PSCustomObject]$TagMapping,

        [Parameter(Mandatory = $true, ValueFromPipeline)]
        [String[]]$VM
    )

    Begin {
        If ($global:DefaultVIServers.count -eq 0) {
            Throw "You are not currently connected to any servers. Please connect first using a Connect cmdlet."
        }
    }

    Process {
        
        ForEach ($SingleVm in $VM) {
            $VmToProcess = Get-VM -Name $SingleVm -Verbose:$false

            $appInfo = $VmToProcess.ExtensionData.Config.ExtraConfig | Where-Object { $_.key -eq "guestinfo.appInfo" }
            If ($appInfo.value -eq '' -or $null -eq $appInfo.value ) {
                Write-Verbose -Message "$VmToProcess : appInfo NOT available"
            } Else {
                Write-Verbose -Message "$VmToProcess : appInfo available"
                $VMappInfo = $appInfo.value | ConvertFrom-Json

                ForEach ($TagMap in $TagMapping) {
                    If ($VMappInfo.applications.a -contains $TagMap.Process){
                        Write-Verbose "$VmToProcess : Found matching application $($TagMap.Process)"
                        $Tag = Get-Tag -Name $TagMap.Tag -Verbose:$false
                        New-TagAssignment -Tag $Tag -Entity $VmToProcess -Verbose:$false
                    }
                }
            }
        }
    }
}
{% endhighlight %}

The tags need to already exist in vCenter.

### How to use Set-ApplicationTag

For ease of explanation I'm going to show how you can assign a tag PowerShell when you find powershell.exe running in a VM.

Based on a simple set of mappings from Process to Tag which can be easily kept in a CSV file -
TagMappings.csv
Process,Tag
mssqlserver.exe,MS SQL Server
oracle.exe, Oracle DB
tomcat6.exe, Tomcat
powershell.exe,PowerShell
iexplore.exe,Internet Explorer

The tags need to already exist in vCenter.

The VM starts with no tags assigned.

![No Tags]({{ site.baseurl }}/images/VMTools-appinfo-tags-notags.png)

Then we load the tag mapping and run the script.

![Run Script]({{ site.baseurl }}/images/VMTools-appinfo-tags-runscript.png)

And check the VM again and see that the PowerShell tag has been assigned to the VM.

![PowerShell Tag]({{ site.baseurl }}/images/VMTools-appinfo-tags-pstag.png)