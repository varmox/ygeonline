---
title: "Quicktip: Tanzu & HA-Proxy" # Title of the blog post.
date: 2023-03-08T20:43:03+02:00 # Date of post creation.
description: "Quicktip: Tanzu & HA-Proxy." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
usePageBundles: false # Set to true to group assets like images in the same folder as this post.
#featureImage: "/vsphere-tig.png" # Sets featured image on blog post.
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
  - VMware 
  - vSphere
  - VMware Tanzu
# comment: false # Disable comment if false.
---

# Quick Tip: vSphere with Tanzu an HA-Proxy

This is a short article with tips and tricks I experienced when deploying vSphere with Tanzu and HA-Proxy as a Load Balancer.

## Deploying Ha-Proxy OVA

**Network**
 
Alwasys use three NICs when deploying in a production environment. It is very important that these  IP addresses do not overlap with the address ranges of your workloads and load balancers, or with the gateway itself.

At first it can be somewhat confusing what networks are needed and which services will be deployed in each network.

- Management: Your Management Network (must be only accessible for Admins, like vCenter)
- Workload: This is the workload IP address for the HA Proxy server. This subnet will be used to for virtual servers created by HA-Proxy. 
- Frontend: This is the frontend IP address for the HA Proxy server. From this network clients will access resources.

For a simple PoC you can safely use two NIC. Management is seperate and Workload/Frontend will be shared.

**Certificate**

Just leave it blank it will be generated for you.

## Getting the Certificate

From your Workstation run the following command to get the Certificates from HA-Proxy needed for the Workload Management Feature Setup:

```
scp root@haproxyMGMTIP:/etc/haproxy/ca.crt ca.crt && cat ca.crt
```

