---
title: "[Guide] vSphere VM - conflicted PortID" # Title of the blog post.
date: 2024-11-22T07:41:26+01:00 # Date of post creation.
description: "How to fix conflicted PortIDs on vSphere VMs (PortID 'c-')" # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
usePageBundles: false # Set to true to group assets like images in the same folder as this post.
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
#featureImageAlt: 'Description of image' # Alternative text for featured image.
#featureImageCap: 'This is the featured image.' # Caption (optional).
#thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - VMware
tags:
  - vSphere Networking
  - Cisco ACI
# comment: false # Disable comment if false.
---

# VMs with conflicted PortIDs

In certain scenarios it can happend that the Ports tan f a vDS Show Port IDs with the prefix "-c" (eg c-245)
vCenter Server creates a conflict port if multiple virtual machine's point to the same port. 
It does not automatically reconfigure the virtual machine to connect to a regular port if the conflict remains and the virtual machine stays connected to the conflict port.

See: https://knowledge.broadcom.com/external/article/318950/vnetwork-distributed-switch-contains-dvp.html

This issue ussually happens when the distributed vSwitch is out-of sync. I've seen this mostly on Cisco ACI VMM Setups. I strongly do not recommend a Cisco ACI VMM [Integration with vSphere.](https://knowledge.broadcom.com/external/article/324518/vmware-support-for-partner-management-in.html)

Anyway, the issue is there, we need to fix it. The fix is quite easy, disconnect the Network Adapter from the VM, save the VM config, then reconnect again. For this manual process I created some scripts:

## PowerCLI Script - Find VMs with conflicted PortIDs

Newest Version [here](https://github.com/soultecag/powercli-scripts/blob/main/Find-VM-with-dvPort-prefix-c.ps1)

```
# Load the PowerCLI SnapIn and set the configuration
Add-PSSnapin VMware.VimAutomation.Core -ea "SilentlyContinue"
Set-PowerCLIConfiguration -InvalidCertificateAction Ignore -Confirm:$false | Out-Null

# Get the vCenter Server address, username and password as PSCredential
$vCenterServer = Read-Host "Enter vCenter Server host name (DNS with FQDN or IP address)"
$vCenterUser = Read-Host "Enter your user name (DOMAIN\User or user@domain.com)"
$vCenterUserPassword = Read-Host "Enter your password (this will be converted to a secure string)" -AsSecureString:$true
$Credentials = New-Object System.Management.Automation.PSCredential -ArgumentList $vCenterUser,$vCenterUserPassword

# Connect to the vCenter Server with collected credentials
Connect-VIServer -Server $vCenterServer -Credential $Credentials | Out-Null
Write-Host "Connected to your vCenter server $vCenterServer" -ForegroundColor Green

Add-PSSnapin VMware.VimAutomation.Core -ea "SilentlyContinue"
Set-PowerCLIConfiguration -InvalidCertificateAction Ignore -Confirm:$false | Out-Null



$vms = Get-VM


$vmInfo = @()

foreach ($vm in $vms) {
   
    $networkAdapters = Get-NetworkAdapter -VM $vm
    
    foreach ($adapter in $networkAdapters) {
        $portID = $adapter.ExtensionData.Backing.Port.PortKey
        
        # Only process if the portID starts with "c-"
        if ($portID -and $portID.StartsWith("c-")) {
            # Create a custom object with VM and port information
            $vmData = [PSCustomObject]@{
                VMName = $vm.Name
                PowerState = $vm.PowerState
                NetworkName = $adapter.NetworkName
                PortID = $portID
            }
            
            # Add the object to the array
            $vmInfo += $vmData
        }
    }
}


$vmInfo | Export-Csv -Path "C:\source\VMPortInfo.csv" -NoTypeInformation
```



## PowerCLI Script - Find VMs with conflicted PortIDs and fix it

This script then detects VMs with "c-" PortID and does the following:
  - list you all VMs
  - will ask, if it should fix the Issue -> disconnects vNIC, waits 5sec, and reconnects the adapter
  - Option to Ping the VMs afterwards (based on the IPs it reads out from VMware Tools)

Newest Version [here](https://github.com/soultecag/powercli-scripts/blob/main/Find-VM-with-dvPort-prefix-c-with-fix.ps1) 


```
# Load the PowerCLI SnapIn and set the configuration
Add-PSSnapin VMware.VimAutomation.Core -ea "SilentlyContinue"
Set-PowerCLIConfiguration -InvalidCertificateAction Ignore -Confirm:$false | Out-Null

# Get the vCenter Server address, username and password as PSCredential
$vCenterServer = Read-Host "Enter vCenter Server host name (DNS with FQDN or IP address)"
$vCenterUser = Read-Host "Enter your user name (DOMAIN\User or user@domain.com)"
$vCenterUserPassword = Read-Host "Enter your password (this will be converted to a secure string)" -AsSecureString:$true
$Credentials = New-Object System.Management.Automation.PSCredential -ArgumentList $vCenterUser,$vCenterUserPassword

# Connect to the vCenter Server with collected credentials
Connect-VIServer -Server $vCenterServer -Credential $Credentials | Out-Null
Write-Host "Connected to your vCenter server $vCenterServer" -ForegroundColor Green


# Get all VMs with portID starting with "c-"
$problematicVMs = Get-VM | Get-NetworkAdapter | Where-Object { $_.ExtensionData.Backing.Port.PortKey -like "c-*" } | Select-Object -ExpandProperty Parent -Unique

# Print out the VMs with problematic portIDs
Write-Host "VMs with portID 'c-':"
$problematicVMs | ForEach-Object { Write-Host $_.Name }

# Ask for confirmation to fix the issues
$confirmation = Read-Host "Do you want to fix those issues for the following VMs - resulting in 5sec Network Connectivity loss? (yes/no)"

if ($confirmation -eq "yes") {
    foreach ($vm in $problematicVMs) {
        Write-Host "Processing $($vm.Name)..."
        
        # Disconnect the network adapter
        Get-NetworkAdapter -VM $vm | Where-Object { $_.ExtensionData.Backing.Port.PortKey -like "c-*" } | Set-NetworkAdapter -Connected $false -Confirm:$false
        
        # Wait for 5 seconds
        Start-Sleep -Seconds 5
        
        # Reconnect the network adapter
        Get-NetworkAdapter -VM $vm | Where-Object { $_.ExtensionData.Backing.Port.PortKey -like "c-*" } | Set-NetworkAdapter -Connected $true -Confirm:$false
        
        Write-Host "Fixed network adapter for $($vm.Name)"
    }
    
    # List all affected VMs
    Write-Host "Affected VMs:"
    $problematicVMs | ForEach-Object { Write-Host $_.Name }
    
    # Option to read out IP and ping VMs
    $pingOption = Read-Host "Do you want to read out IPs and ping the affected VMs? (yes/no)"
    
    if ($pingOption -eq "yes") {
        foreach ($vm in $problematicVMs) {
            $ip = $vm.Guest.IPAddress[0]
            if ($ip) {
                Write-Host "$($vm.Name) IP: $ip"
                $pingResult = Test-Connection -ComputerName $ip -Count 1 -Quiet
                if ($pingResult) {
                    Write-Host "Ping successful for $($vm.Name)"
                } else {
                    Write-Host "Ping failed for $($vm.Name)"
                }
            } else {
                Write-Host "Unable to retrieve IP for $($vm.Name)"
            }
        }
    }
} else {
    Write-Host "Operation cancelled. No changes were made."
}
```
Hpe this helps fixing the Issue.
