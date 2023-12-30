---
title: "LAN Party Infrastructure - powered by VMware Tanzu"
date: 2023-11-29T22:04:40+01:00
draft: true
featured: true # Sets if post is a featured post, making appear on the home page side bar.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
usePageBundles: false # Set to true to group assets like images in the same folder as this post.
#featureImage: "/vsphere-tig.png" # Sets featured image on blog post.
#featureImageAlt: 'Description of image' # Alternative text for featured image.
#featureImageCap: 'This is the featured image.' # Caption (optional).
thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - IT
  - VMware
  - Kubernetes
tags:
  - Kubernetes 
  - VMware Tanzu
  - VMware Tanzu Kubernetes Grid
# comment: false # Disable comment if false.
---
**Intro**

This blog post is about the infrastructure services that can be used at a LAN party. The following services are provided:

- Caching Server for Steam, Origin, Windows Updates etc (lancache.net) - Clients will be served from that Cache instead of using your Internet Bandwith
- DNS Server
- Various Gameserver running on Tanzu Kubernetes Grid with Agones

We will run those workloaks on top of Tanzu Kubernetes Grid (Free Edition). VMware offers Tanzu Kubernetes Grid in a free Edition instead of Tanzu Community Edition that has been postponed. TKG free Tier can run up to 100 Cores.

***What is Tanzu Kubernetes Grid?***

VMware has multiple Kubernetes Solution all under the VMware Tanzu Name. Tanzu Kubernetes Grid is a Tool to bootstrap your Kubernetes Cluster. TKG ist installed on a Workstation or VM that will provision docker containers, install kind to give you a Webinterface and CLI Tools to bootstrap a Kubernetes Cluster. This Kubernetes Cluster can run on vSphere or AWS and Azure.

***Difference TKG / Tanzu with vSphere***


**Prerequieries**

- VMware vSphere 8
- 
