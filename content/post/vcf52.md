---
title: "VMware Cloud Foundation 5.2"
date: 2024-06-25T20:48:30+02:00
draft: false
description: "VMware Cloud Foundation Updates for Tanzu" # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
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
  - VMWare
  - VCF
tags:
  - VMware
  - VCF
# comment: false # Disable comment if false.
---



The new VCF 5.2 Update delivers quite a few interesting updates. This blog post covers expecially the updates and improvements for Tanzu.

## vSphere with Tanzu Update -> vSphere IaaS Control Plane

vSphere with Tanzu is passé - the new naming is vSphere IaaS Control Plane.

The naming already suggest - not only Kubernetes Cluster can be deployed, also VMs (this was possible a long time but with the new naming the focus is not only on k8s). Via the vm-operator you can deploy VMs alongside Tanzu Kubernetes Cluster. The IaaS Control Plane is really interesting as you can deploy VMs with Code. A YAML File will describe your VM. It is even possible to deploy Windows Workloads via VM-Operator with sysprep! (blog to follow)

Deployment of Windows Server within a ArgoCD Pipeline is possible (if you would want that ;))

## LCI - Local consumption Interface

With the new Local Consumption Interface within vSphere UI you can deploy Tanzu Kubernetes Clusters or VMs - both via IaaS Control Plane with a GUI!

Surely some of you already knew the Cloud Consumption Interface (CCI) within Aria - Local Cloud Consumption Interface is a built in UI within vSphere UI (no need to deploy Aria)


![vSphere LCI](https://github.com/varmox/ygeonline/blob/main/static/images/vcf5.2-lci.png?raw=true)

## Tanzu Kubernetes Cluster Autoscaling

As the name already suggest: Autoscaling of Tanzu Kubernetes Workers Nodes (VMs)! Only requirements is to have TKR Relase 1.25.