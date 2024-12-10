---
title: "Vsan 2 Node" # Title of the blog post.
date: 2024-12-10T20:20:23+01:00 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: true # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
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
  - Technology
tags:
  - Tag_name1
  - Tag_name2
# comment: false # Disable comment if false.
---

# vSAN 2-Node Cluster Guide and Recommendations

This Guide

## Assumptions

The following assumptions where made when creating this blog post.

- You have vSAN Ready Nodes or all Hardware is on the vSAN [Hardware Compatibility List](https://compatibilityguide.broadcom.com/).
- You have VMware Cloud Foundation (VCF) Licenses or VVF with enough vSAN Add-on Licences
- Phyiscal Network Adapter for vSAN have a Bandwith of 25Gbits or above
- General Knowledge of vSAN and vSphere

# Design Considerations

## General 

- Do you have a dedicated vCenter for your 2-Node Cluster?
- Where do you run the vCenter VM? On the 2-Node Cluster itself or outside?

## Network

### vSAN Traffic: direct attached or Switched

With a 2-Node Cluster you have the possibility to run vSAN (and vMotion) traffic over direct attached Cables. This is a great solution for RoBo or Site which do not offer 25Gbits Ports on Network Switches.

Just consider the following:

- Expanding your vSAN 2-Node Cluster to a 3-Node Cluster can be a little bit tricky.
- 

## Maintenance Scenario

When you want to Update your ESXi Hosts there are a few specialities in 2 2-Node vSAN Cluster Setup:

- Full data migration is not possible when entering maintenance mode.
- You must use the "Ensure accessibility" option when putting a host into maintenance mode

When using Lifecycle Manager (vLCM) with Cluster Images the Remedation Process will not put Hosts into Maintenance Mode.
Also Maintenance Mode will not evacuate any Workloads. Why is that?

Well a 2-Node vSAN Cluster will only support the VM Storage Policy of Mirror/RAID1. 

To perform maintenance on a 2-node vSAN cluster:
- Manually vMotion VMs to the other host.
- Put the host into maintenance mode using "Ensure accessibility" option.
- Perform the required maintenance
