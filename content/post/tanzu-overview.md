---
title: "VMware Tanzu - Overview" # Title of the blog post.
date: 2023-02-08T20:43:03+02:00 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: true # Sets whether to render this page. Draft of true will not be rendered.
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
  - Kubernetes
tags:
  - VMware 
  - VMware Tanzu
  - Kubernetes

# comment: false # Disable comment if false.
---
# Tanzu Kubernetes Grid vs vSphere with Tanzu #

Lately had I has internal and discussion with customers about Tanzu. There seems to be some confusing about the Tanzu Portfolio and what Tanzu Product is best for specific use cases.

## Tanzu Kubernetes Grid (TKG) ##

TKG is a solution that bootstraps kubernetes clusters to diffrent cloud providers. First a management cluster (based on docker and kind) will be created from a bootstrap workstation which then itself provides a Webinterface and additional CLI Tools to create your kubernetes cluster to:

- vSphere
- AWS
- Azure

The management cluster will provision the VMs on the target and install & configure kubernetes within those VMs.

When vSphere is used as a Target there is no integration of TKG with vSphere UI. Just a bunch of VMs. 

## vSphere with Tanzu aka Workload Management ##

In newer version of vCenter there is the option of "Workload Management" within the vSphere Burger Menu. Workload Management is nothing other than vSphere with Tanzu. With vSphere with Tanzu obviously only works with vSphere. Kubernetes Ressources are provisioned on a vSphere Cluster. The first step is to create a Supervisor Cluster, which consists of three VMs in vSphere. Kubernetes itself is then provisioned with kubeadm - so it isn't a specialized version of kubernetes - plain vanilla kubernetes will be provided.

If the supervisor cluster then is created, additional vSphere Namespaces need to be provisioned (vSphere Name Space isn't the same as a Kubernetes Namespace!) Within that Namespace then finally a Tanzu Kubernetes Cluster (inofficially TKC) can be created. The Tanzu Kubernetes Cluster itself consist of a minimum of 1 Master VM and 1 Worker VM - Best Practise is of course 3 Master VMs and 2 or more Worker VMs.

**vSphere with Tanzu minimum Ressources**

- 3 Supervisor VMs
    - small nodes: 


