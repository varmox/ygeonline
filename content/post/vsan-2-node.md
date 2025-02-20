---
title: "[Guide] 2-Node vSAN Cluster" # Title of the blog post.
date: 2024-12-10T20:20:23+01:00 # Date of post creation.
description: "Examples and Design Considerations for a 2-Node vSAN Cluster" # Description used for search engine.
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
codeMaxLines: 50 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - VMware
tags:
  - vSAN
  - vSphere
# comment: false # Disable comment if false.
---

# vSAN ESA 2-Node Cluster Guide and Recommendations

This Guide will help you in your Design Considerations and during the Deployment of a 2-Node vSAN Cluster. 

## Assumptions

The following assumptions where made when creating this blog post.

- You have vSAN Ready Nodes or all Hardware is on the vSAN [Hardware Compatibility List](https://compatibilityguide.broadcom.com/).
- You have VMware Cloud Foundation (VCF) Licenses or VVF with enough vSAN Add-on Licences
- Phyiscal Network Adapter for vSAN have a Bandwith of 25Gbits or above
- General Knowledge of vSAN and vSphere

# Design 

## Constraints

- Only one host failure can be tolerated at a time (FTT1)
- Adding nodes beyond two would require reconfiguration to a standard vSAN cluster.
- Requires a witness host, which needs two IP addresses - one for management and one for vSAN traffic
- Managing multiple 2-node clusters can increase operational complexity

## General Considerations

- Do you have a dedicated vCenter for your 2-Node Cluster?
- Where do you run the vCenter VM? On the 2-Node Cluster itself or outside?
- Does the two physical Server run on the same physical site?



### vSAN Traffic: direct attached or Switched

With a 2-Node Cluster you have the possibility to run vSAN (and vMotion) traffic over direct attached Cables. This is a great solution for RoBo or Site which do not offer 25Gbits Ports on Network Switches.

Just consider the following:

- Expanding your vSAN 2-Node Cluster to a 3-Node Cluster can be a little bit tricky.
- If your setup vMotion vmk on the direct attached links, plan you migration path. If you want to migrate from other vSphere Clusters, additional vMotion vmks should be created (which have a L3 Uplink to your network).

#### vSAN Witness

If you go directed attached you need a addtional vmk that handels the traffic to the vSAN Witness Appliance.

- vSAN Witness Appliance should NOT run inside the 2-Node vSAN Cluster
- Have a L3 or L2 Network for vSAN Witness traffic to your vSphere Cluster which hosts the vSAN Witness Appliance.
- If you have a dedicated vCenter for your 2-Node vSAN Cluster: the Witness Appliance is added to a seperate Cluster (Witness Cluster) to the 2-Node vSAN vCenter.

## Network: Physical and vmkernel Adapter Example

The following graphics is an example of a 2-Node vSAN Cluster:

```
Physical NICs    VDS Switches    VMkernel Interfaces
+----------+     +-----------+   +----------------------+
| vmnic0   |-----| FRONTEND  |---| vmk0 (Management)    |
+----------+    /| VDS       |   | vmk1 (vSAN Witness)  |
| vmnic2   |---/ +-----------+  +----------------------+
+----------+                    
                                 
+----------+     +-----------+    +----------------------+
| vmnic1   |-----| BACKEND   |-----| vmk2 (vSAN Data)     |
+----------+    /| VDS       |\    | vmk3 (vMotion)       |
| vmnic3   |---/ +-----------+ \   +----------------------+
+----------+
```

Per Hosts we have four physical Network Adapter:

- 2x10Gbits Uplinks for Frontend Traffic (Workload, vSAN Witness) - those could also be 1Gbit links, depending on the Workloads
- 2x25Gbits directed attached Links for vSAN and vMotion Traffic

## Distributed Switch and Distributed Portgroup:

**FRONTEND VDS**
| Portgroup | vmk Service | MTU | Teaming and Failover |
| -------- | -------- |-------- |-------- |
| vpg_management  | Management   | 1500 | Route based on physical NIC load |
| vpg_vsan-witness | vSAN Witness   | 1500 | Route based on physical NIC load |


**BACKEND VDS**

| Portgroup | vmk Service | MTU | Teaming and Failover |
| -------- | -------- |-------- |-------- |
| vpg_vsan-data   | vSAN   | 9000 | Route based on physical NIC load |
| vpg_vmotion  | vMotion   | 9000 | Route based on physical NIC load |

For IP and VLAN assigned I suggest you reserve the VLAN + IP-Ranges within your companies IPAM, a) IP Adress conflict happens b) you have it documented.

## Maintenance Scenario

When you want to Update your ESXi Hosts there are a few specialities in 2-Node vSAN Cluster Setup:

- Full data migration is not possible when entering maintenance mode.
- You must use the "Ensure accessibility" option when putting a host into maintenance mode

When using Lifecycle Manager (vLCM) with Cluster Images the Remedation Process will not put Hosts into Maintenance Mode.
Also Maintenance Mode will not evacuate any Workloads. Why is that?

Well a 2-Node vSAN Cluster will only support the VM Storage Policy of Mirror/RAID1. An DRS want to ensure that the virtual object will be in sync. Iif a Hosts is down in a 2-Node vSAN Cluster, your virtual Objects will always be affected.

To perform maintenance on a 2-node vSAN cluster:
- Manually vMotion VMs to the other host.
- Put the host into maintenance mode using "Ensure accessibility" option.
- Perform the required maintenance

## Performance

Performance Benchmarking should be done with [HCIBench](https://github.com/vmware-labs/hci-benchmark-appliance).

**Example**

Setup:
- 2x HPE DL365 Gen11
- 25Gbit DAC between the Hosts
- 7x7,68Tb Samsung PM9A3 SSD

![vSAN 2 Node direct attached Cluster](https://i.imgur.com/ZfPSSSN.png)

