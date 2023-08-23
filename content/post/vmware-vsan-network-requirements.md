---
title: "VMware vSAN Network Considerations" # Title of the blog post.
date: 2023-08-23T23:13:14+02:00 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
usePageBundles: false # Set to true to group assets like images in the same folder as this post.
featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
#featureImageAlt: 'Description of image' # Alternative text for featured image.
#featureImageCap: 'This is the featured image.' # Caption (optional).
thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Technology
tags:
  - VMware
  - vSAN
  - vSAN over RDMA
# comment: false # Disable comment if false.
---

**General**

- 10 GbE network switches are the minimum requirement for all-flash vSAN Clusters
- 25 GBE network switches are recommended
- Switches that support higher port speeds are designed with higher Network Processor Unit (NPU) buffers. An NPU shared switch buffer of at least 16 MB is recommended for 10 GbE network connectivity. An NPU buffer of at least 32 MB is recommended for more demanding 25 GbE network connectivity.

With vSAN ESA 25GbE Adapters are a must. (In ESA-AF-0 vSAN Ready Nodes the network bandwidth has been mentioned as 10 GbE - so probalby 10GbE will work for smaller ESA Deployments)

**vSAN over RDMA**

With vSphere 7.0 U2 vSAN over RDMA is supportet. RDMA typically has lower CPU utilization and less I/O latency. 

If your vSAN cluster will include adapters that support RoCE (RDMA over Converged Ethernet) for vSAN storage connectivity, the supporting network must support a “lossless” transport. A “lossless” network is defined as one where no frames are dropped because of network congestion.

- Select switches that support Data Center Bridging (DCB). The Data Center Bridging feature supports the elimination of packet loss due to buffer or queue overflow.
- The Data Center Bridging must support bandwidth allocation based on priority settings, known as Class of Service (CoS).
- Priority Flow Control (PFC) is required on the switches to provide RoCE traffic a higher priority than other network traffic.


All the nodes in the cluster connected to the common vSAN datastore must be configured with RoCE-supported adapter cards of the same vendor and the same model. This is strongly encouraged as a best practice to remove any possibility of slight variances in adapters from different vendors or different models disrupting I/O on the vSAN datastore.
- The Ethernet ports on the RoCE-supported adapters are reserved for vSAN traffic only.
- The physical network supporting vSAN is configured as a “lossless” network.
