---
title: "VMware vCenter Troubleshooting Guide" # Title of the blog post.
date: 2024-08-23T22:24:18+02:00 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: true # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
usePageBundles: false # Set to true to group assets like images in the same folder as this post.
featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
featureImageAlt: 'Description of image' # Alternative text for featured image.
featureImageCap: 'This is the featured image.' # Caption (optional).
thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - VMware
tags:
  - vCenter
  - VCF
# comment: false # Disable comment if false.
---


# vCenter Troubleshooting Guide 

The following Troubleshooting Guide applies to vCenter 8 and above. 

Create a VM Snapshot of your vCenter before proceeding with any step!

## Log Files

All vCenter Log Files are stored under:

```
/var/log/vmware/
```

vCenter logs are grouped by component and purpose:
- vpxd.log - Main vCenter Server log for client connections, tasks, and host communication5
- vpxd-profiler.log - Profiled metrics for vCenter operations
- eam.log - ESX Agent Manager logs
- sms.log - Storage Monitoring Service logs
- ls.log - Licensing Services logs

Some key log files and directories include:

- /var/log/vmware/vpxd/vpxd.log - The main vCenter Server log
- /var/log/vmware/vsphere-ui/ - vSphere UI logs
- /var/log/vmware/eam/eam.log - ESX Agent Manager log

Additional Important Logs
- /var/log/vmware/vpostgres/ - VMware Postgres service logs
- /var/log/vmware/vcha/ - vCenter High Availability service logs
- /var/log/vmware/rhttpproxy/ - VMware HTTP Reverse Proxy service logs
- /var/log/vmware/content-library/ - VMware Content Library Service logs

## Networking

### Manual Network Config

Run the following command to change Networking Settings from BASH

```
/opt/vmware/share/vami/vami_config_net
```

### DNS

***Manual DNS Config***


***Flush DNS***

```

systemctl restart systemd–resolved.service

systemctl restart dnsmasq

```

### HTTP Proxy

Log File: /var/log/vmware/rhttpproxy/

To set a HTTP Proxy run:

```
/opt/vmware/share/vami/vami_config_net
```

It is also possible to edit the file directly:

```
vi /etc/sysconfig/proxy
```

## Certificate Management


Log File: /var/log/vmware/vmcad/certificate-manager.log


### Renew all vCenter Machine Certificates

Generally this should be done of all VMCA Certificates are self-signed and are expired. Later you can change the certificates to named ones from your Enterprise PKI.

Run vCenter Certificate Manager:

```
/usr/lib/vmware-vmca/bin/certificate-manager
```

Choose your desired option. Most commonly option 4 or 8 are used.

This step automatically restarts the vCenter Server services. Additionally, the Name, Hostname, and VMCA values should match the Primary Network Identifier (PNID). The PNID should always match the Hostname. 

To get the vCenter PNID:

```
/usr/lib/vmware-vmafd/bin/vmafd-cli get-pnid --server-name localhost
```


# vLCM

***Cannot sync depot***

```

/var/log/vmware/vmware-updatemgr/vum-server/vmware-vum-server.log

```
A usual error is that vCenter is unable to reach vmwaredepot. Check DNS, Firewall and HTTP Proxy Settings. The log file indicated it the following:

```
-->     "error_type": "ERROR",
-->     "messages": [
-->         {
-->             "args": [],
-->             "default_message": "A depot is inaccessible or has invalid contents. Make sure an official depot source is used and verify connection to the depot.",
-->             "id": "com.vmware.vcIntegrity.lifecycle.depotContent.ValidationError"
-->         }
-->     ]
--> }"
--> }
```
Also check your connected depots. Maybe there is a old depot still configured (like a old HPE OneView Instance).

# Restore 

If during a File Level Restore something went run, check your logs at:



# vCenter and ESXi 

**Disconnect ESXi if vCener no longer exists**

```
cmsso-util unregister --node-pnid vcenter.domain.com --username [administrator@vsphere.local](mailto:administrator@vsphere.local) --passwd pw
```
