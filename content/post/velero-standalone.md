---
title: "Velero" # Title of the blog post.
date: 2024-11-30T00:46:45+01:00 # Date of post creation.
description: "Velero with vSphere Kubernetes Service." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: true # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
usePageBundles: false # Set to true to group assets like images in the same folder as this post.
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
#featureImageAlt: 'Description of image' # Alternative text for featured image.
#featureImageCap: 'This is the featured image.' # Caption (optional).
#thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 30 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - VMware
  - Kubernetes
tags:
  - vSphere Kubernetes Service
  - Velero
# comment: false # Disable comment if false.
---

# Velero 

This blog post explains how to install velero on different configurations on vSphere with Tanzu / IaaS Control Plane.


## Different Velero Installations with TGKS

Depending on your Supervisor Cluster Configuration, there are different ways to install Velero. Each approach offers different usage on velero.

### Ways to Install Velero

- Velero Plugin for vSphere
- Velero "Standalone" via Helm Chart
- Velero "Standalone" via velero-cli





