---
title: "[Guide] VMware vSAN Network Considerations" # Title of the blog post.
date: 2023-08-23T23:13:14+02:00 # Date of post creation.
description: "Article description." # Description used for search engine.
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
  - Technology
tags:
  - VMware
  - vSAN
  - vSAN over RoCE
# comment: false # Disable comment if false.
---

# Intro 
VMware vSAN is a Hyperconverged Infrastructure Solution from VMware. It basically turns your servers direct attached disks into a storage cluster.

## Minimum Bandwidth Requirements vSAN ESA

- A minimum bandwidth of 25 Gbps is required, 10Gbit works but is only supported for the AF0 Config.
- For larger clusters or environments with high-capacity NVMe disks, even 25 Gbps may be insufficient. If you can do 100Gbps, the Performance gain will be massive.
- vSAN MAX Requires 100 Gbps

- Switches that support higher port speeds are designed with higher Network Processor Unit (NPU) buffers. An NPU shared switch buffer of at least 16 MB is recommended for 10 GbE network connectivity. An NPU buffer of at least 32 MB is recommended for more demanding 25 GbE network connectivity.



## Network Configuration

- All hosts in the vSAN cluster must be connected to a vSAN Layer 2 or Layer 3 network
- LACP can be used, active/passive connection is recommended due to operational complexity.

## Latency Requirements

- Single Site vSAN Cluster Requires less than 1 ms RTT (Round Trip Time) inter-node latency
- For vSAN streched Clusters RTT must be below 5ms for the inter-site communication and less than 1 ms RTT within each site

### vSAN Strechted Cluster - Witness

- 2 Mbps per 1000 components (Maximum of 100 Mbps with 45k components) bandwidth between nodes and vSAN Witness Hosts

## General Considerations

- Use a dedicated VLAN on the Physical Network Switches for vSAN Traffic. If possible use Quality of Service Features to priorize vSAN traffic.
- Use two physical Uplinks for vSAN Traffic, do not mix with VM Workload traffic. 
- Use a dedicated virtual Distributed Switch for vSphere Backend Services (vSAN, vMotion, FT)
- Use Network I/O Control 

## vSAN over RoCE

With vSphere 7.0 U2 vSAN with RDMA is supported. RDMA typically has lower CPU utilization and less I/O latency. 

If your vSAN cluster will include adapters that support RoCE (RDMA over Converged Ethernet) for vSAN storage connectivity, the supporting network must support a “lossless” transport. A “lossless” network is defined as one where no frames are dropped because of network congestion.

- Select switches that support Data Center Bridging (DCB). The Data Center Bridging feature supports the elimination of packet loss due to buffer or queue overflow.
- The Data Center Bridging must support bandwidth allocation based on priority settings, known as Class of Service (CoS).
- Priority Flow Control (PFC) is required on the switches to provide RoCE traffic a higher priority than other network traffic.


All the nodes in the cluster connected to the common vSAN datastore must be configured with RoCE-supported adapter cards of the same vendor and the same model. This is strongly encouraged as a best practice to remove any possibility of slight variances in adapters from different vendors or different models disrupting I/O on the vSAN datastore.
- The Ethernet ports on the RoCE-supported adapters are reserved for vSAN traffic only.
- The physical network supporting vSAN is configured as a “lossless” network. Look for RoCEv2 support with your Network Switch Vendor.

Supported RDMA Network adapters can be found here: 

https://www.vmware.com/resources/compatibility/search.php?deviceCategory=rdmanic&details=1&vsan_type=rdmanic&page=1&display_interval=10&sortColumn=Partner&sortOrder=Asc

