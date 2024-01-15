---
title: "VMware Tanzu Kubernetes Grid -  getting started"
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
# Intro

This blog post is about getting started with VMware Tanzu Kubernetes Grid (TKG). TKG is used to deploy Tanzu Kubernetes Cluster to various Cloud Providers including vSphere, AWS and Azure. 
First, a management cluster (based on Docker and kind) is created from a Bootstrap Workstation, which itself then  provides its web interface and additional CLI tools to create a Kubernetes cluster.

TKG has no integration in vSphere UI. If you wan't to integrate Tanzu with vCenter/vSphere UI - use vSphere with Tanzu instead.

## Install Tanzu Kubernetes Grid


**Prerequieries**

- VMware vSphere 8 (7 U3 should work, in this tutorial we are using vSphere 8)
- Bootstrap VM or Workstation with Docker
- Access to VMware Customer Connect

## TKG Bootstrap Machine on Fedora ##

See [this blog] (https://ygerber.online/post/tanzu-kubernetes-grid-workstation-setup/) post to create a Bootstrap Machine based on Fedora

## Create Management Cluster 

Create the Management Cluster on your Machine: