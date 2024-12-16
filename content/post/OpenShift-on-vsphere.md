---
title: "[Guide] OpenShift on vSphere - Roles and Permissions" # Title of the blog post.
date: 2024-11-19T23:20:09+01:00 # Date of post creation.
description: "Quick Guide to create the needed Roles for OpenShift on vSphere (IPI)." # Description used for search engine.
featured: false # Sets if post is a featured post, making appear on the home page side bar.
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
  - VMware
tags:
  - vSphere
  - OpenShift
# comment: false # Disable comment if false.
---

# OpenShift Cluster on vSphere - Roles and Permissions

## Deployment Mode: Installed Provisioned Infrastructure

 OpenShift Installer-Provisioned Infrastructure (IPI) is a deployment method for OpenShift Container Platform that provides a full-stack automated installation and setup process.

 The installer manages all aspects of the cluster deployment, including the underlying infrastructure and the operating system itself. IPI creates a bootstrap virtual machine on a provisioner node, which assists in deploying the OpenShift cluster.

 The most common way to deploy OpenShift is on vSphere. For the vSphere Integration Permissions are needed:

 ## Create vSphere Roles and Permissions

I created a PowerShell Script to create all the necessary vSphere Roles and Permissions:

 Full script [here](https://github.com/soultecag/powercli-scripts/blob/main/vSphere-Openshift-IPI-vSphere-Roles.ps1)

The script will generate the Roles, assigning the vSphere Roles to the required vSphere Objects is still needed, depending on your setup.

Always needed:

- Assing the 'OpenShift_vCenter Role' on the vCenter Object.
- Assign the 'OpenShift_Datastore Role' to the Datastore OpenShift will run on.
- Assign the 'OpenShift_PortGroup Role' to the Portgroup OpenShift will run on.
- Assign the 'OpenShift_VMFolder Role' where your OpenShift VMs will reside

Optional:

- Assign the 'OpenShift_Cluster Role' to your vSphere Cluster. If VMs will be created in the cluster root
- Assign the 'OpenShift_ResourcePool Role' to your vSphere Ressource Pool. If an existing resource pool is provided
- Assign the 'OpenShift_Datacenter Role' to your Datacenter. If the installation program creates the virtual machine folder


### Additional Information

- [OpenShift on vSphere Requirements](https://docs.openshift.com/container-platform/4.17/installing/installing_vsphere/ipi/ipi-vsphere-installation-reqs.html)
- [PowerCLI Script for vSphere Role Creation](https://github.com/soultecag/powercli-scripts/blob/main/vSphere-Openshift-IPI-vSphere-Roles.ps1)


