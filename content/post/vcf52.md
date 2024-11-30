---
title: "[WhatsNew] VMware Cloud Foundation 5.2"
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
  - VMware
tags:
  - VMware
  - VMware Cloud Foundation
  - VMware Tanzu
# comment: false # Disable comment if false.
---



The new VCF 5.2 Update delivers quite a few interesting updates. This blog post covers expecially the updates and improvements for Tanzu.

Part 1 covers new Feature within Tanzu on vSphere 8 Update 3.

## vSphere with Tanzu Update -> vSphere IaaS Control Plane

vSphere with Tanzu is passé - the new naming is vSphere IaaS Control Plane. 

The naming already suggest - not only Kubernetes Cluster can be deployed, also VMs (this was possible a long time but with the new naming the focus is not only on k8s). Via the vm-operator you can deploy VMs alongside Tanzu Kubernetes Cluster. The IaaS Control Plane is really interesting as you can deploy VMs with Code. A YAML File will describe your VM. For that the VM Class now support much more configuration, it handled like a normal VM provisioned through vSphere UI.

It is even possible to deploy Windows Workloads via VM-Operator with sysprep! (blog to follow)

Deployment of Windows Server within a ArgoCD Pipeline is possible (if you would want that ;))

Now also the Backup via vSphere Storage APIs for Data Protection (VADP) for VMs deployed via vm-operator are supportet (as "normal" VMs) - within Veeam you can just backup the whole vSphere Namespace (with is basically a advanced vSphere Ressource Pool)

## LCI - Local consumption Interface

With the new Local Consumption Interface within vSphere UI you can deploy Tanzu Kubernetes Clusters or VMs - both via IaaS Control Plane with a GUI!

Surely some of you already knew the Cloud Consumption Interface (CCI) within Aria - Local Cloud Consumption Interface is a built in UI within vSphere UI (no need to deploy Aria)


![vSphere LCI](https://github.com/varmox/ygeonline/blob/main/static/images/vcf5.2-lci.png?raw=true)

#### LCI Installation

LCI is a supervisor Service. Supervisor Services are deployed directly to the Supervisor instead of deploying to a TKGS Cluster.

- https://docs.vmware.com/en/VMware-vSphere/8.0/vsphere-with-tanzu-services-workloads/GUID-4843E6C6-747E-43B1-AC55-8F02299CC10E.html
- https://vsphere-tmm.github.io/Supervisor-Services/#consumption-interface

Download Link of LCI YAML: https://vmwaresaas.jfrog.io/ui/native/supervisor-services/cci-supervisor-service/v1.0.0/cci-supervisor-service.yml 

## vSAN streched Cluster support for Tanzu Kubernetes Grid Service (TKGs)

With vSphere 8 Update 3 TKGS now supports vSAN streched Clusters. Interesting is that Kubernetes Control Plane Nodes and the Supervisor Control Plane Nodes should be kept at one site within a vSAN streched Clusters. This is due to etcd. etcd  requires more than half of the replicas to be available at any time.

- The 3 Supervisor Control Plane VMs should be placed in the same site
- All the Control Plane VMs of any given TKGs cluster should be placed in the same site
- Worker Nodes can be streched acroos two sites

## Tanzu Kubernetes Cluster Autoscaling

As the name already suggest: Autoscaling of Tanzu Kubernetes Workers Nodes (VMs)! Only requirements is to have TKR Relase 1.25


#### Links

[1] https://blogs.vmware.com/cloud-foundation/2024/06/25/vmware-cloud-foundation-launch/

[2] https://core.vmware.com/resource/whats-new-vsphere-update-3-vsphere-iaas-control-plane

[3] https://docs.vmware.com/en/VMware-vSphere/8.0/rn/vmware-vsphere-with-tanzu-80-release-notes/index.html