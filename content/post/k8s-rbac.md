---
title: "RBAC on VKS" # Title of the blog post.
date: 2024-12-08T12:29:05+01:00 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: true # Controls if a table of contents should be generated for first-level links automatically.
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
  - Kubernetes
  - VMware
tags:
  - vShere Kubernetes Service
  - Tanzu Kubernetes
# comment: false # Disable comment if false.
---

# RBAC on vShere Kubernetes Service

Role-based Access Control on vSphere Kubernetes Cluster has a few Keypoint which should be though about. The main question you have to ask yourself: Which Level of access do I want to grant to which people?

# Option 1: vSphere Namespace Permissions

With vSphere Namespace Permissions ([Select Permissions > Add Permissions](https://docs.vmware.com/en/VMware-vSphere/8.0/vsphere-with-tanzu-tkg/GUID-223D91FB-C4CB-4DA7-8B3F-24721ABDFBC7.html)) you grant access directly on the vSphere Namespace via vCenter.

| Option   | Description                                                                                                                                  |
|----------|----------------------------------------------------------------------------------------------------------------------------------------------|
| Can View | Can read TKG cluster objects in the vSphere Namespace. No permissions mapped to Kubernetes roles. See Role Permissions and Bindings.         |
| Can Edit | Can create, read, update, and delete TKG cluster objects in the vSphere Namespace. Can operate TKG clusters provisioned in the vSphere Namespace as the Kubernetes cluster-admin. See Role Permissions and Bindings. |
| Owner    | Same permissions as Can Edit, with the additional permission to create and manage vSphere Namespaces using kubectl. Only available with vCenter SSO. See Role Permissions and Bindings. |



This means - depending on the Permission you give:


- User has Access to all Ressources within that vSphere Namespace:
  - all TKGS/VKS Guest Clusers (kubectl)
  - all VMs wihtin that Namespace
- Create new TKGS/VKS Clusters
- Create new VMs


