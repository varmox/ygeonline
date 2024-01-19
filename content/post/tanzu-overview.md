---
title: "VMware Tanzu - Overview" # Title of the blog post.
date: 2023-02-08T20:43:03+02:00 # Date of post creation.
description: "Article description." # Description used for search engine.
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
  - Kubernetes
tags:
  - VMware 
  - VMware Tanzu
  - Kubernetes

# comment: false # Disable comment if false.
---
# Tanzu Portfolio explained #

Lately, we've been having lots conversations internally and with customers about Tanzu.
There appears to be some confusion about Tanzu's portfolio and which Tanzu products are best suited for specific use cases.

## Kubernetes Runtimes

To actually run Kubernetes Clusters with Tanzu, the following Products are available:

- Tanzu Kubernetes Grid
- vSphere with Tanzu

## Tanzu Kubernetes Grid (TKG) ##

TKG is a solution for routing Kubernetes clusters to different cloud providers.
First, a management cluster (based on Docker and kind) is created from a Bootstrap Workstation, which itself then  provides its web interface and additional CLI tools to create a Kubernetes cluster.

The following providers are supported:

- vSphere
- AWS
- Azure

The management cluster will provision the VMs on the target and install & configure kubernetes within those VMs.

When vSphere is used as a Target there is no integration of TKG with vSphere UI. Just a bunch of VMs will be provisioned.

If you want to deploy your Kubernetes Cluster and Integration with vCenter and other vSphere Services. Look at vSphere with Tanzu. It has also been built with the vSphere Administrator in mind.

If you want to deploy your Kubernetes Cluster to Azure or Amazon Webservices, use Tanzu Kubernetes Grid.


### TKGm

### TKGI


## vSphere with Tanzu aka Workload Management ##

In newer version of vCenter there is the option of "Workload Management" within the vSphere Burger Menu. Workload Management is the same as vSphere with Tanzu. With vSphere with Tanzu obviously only works with vSphere. Kubernetes Ressources are provisioned on a vSphere Cluster. 

The first step is to create a Supervisor Cluster, which consists of three VMs in vSphere. Kubernetes itself is then provisioned via the TKG Service running on the Supervisor Cluster.


If the supervisor cluster then is created, additional vSphere Namespaces need to be provisioned (vSphere Namespace isn't the same as a Kubernetes Namespace!) Within that Namespace then a Tanzu Kubernetes Cluster can be created with the TKG Service. The Tanzu Kubernetes Cluster (TKC) are upstream aligned, fully conformant Kubernetes Cluster. It is also possible to provision VMs withing a vSphere Namespace. The Idea is, that a vSphere Administrator isolates Workloads through vSphere Namespaces that then can be used by DevOps/Platform Operators to provision their Kubernetes Clusters.

User workloads then are running inside a Tanzu Kubernetes Cluster.

**vSphere with Tanzu minimum Ressources**

- vSphere Cluster with DRS
- 3 Supervisor VMs
- NSX-T or vSphere Distributed Switches
-   vDS must use NSX-ALB or HA-Proxy as an external Load Balancer - NSX-ALB is recommended.

## Tanzu Mission Control 

Tanzu Mission Control is a Tool to operate your Tanzu Kubernetes Clusters. Besides Tanzu Kubernetes Cluster it can also managed Azure Kubernetes Service (AKS) and Elasic Kubernetes Service (EKS) from Amazon. It either can be deployed on-prem (TMC self-hosted) with some limitations or consumed as a SaaS Service.

**TMC self-hosted Requirements**

- vSphere with Tanzu
- Supervisor Services:
  - Contour
  - Harbor Image Registry

