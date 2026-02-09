---
title: "Data Services Manager - K8s " # Title of the blog post.
date: 2026-02-09T17:40:28+01:00 # Date of post creation.
description: "Article description." # Description used for search engine.
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
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - VMware
  - DSM
tags:
  - DSM
  - Data Services Manager
  - PostgreSQL
# comment: false # Disable comment if false.
---
# VMware Data Services Manager - K8s Operator

VMware DSM provisiones Data Services like Postgres, MySQL and MSSQL. You can consume DSM through UI (DSM UI or VCF Automation) or via K8s Operator. 

# Scenario 1 - Postgres HA Cluster

We have the scenario that we have a kubernetes cluster and some apps need a PostgreSQL Database. In general it is a good idea to have your persitent data outside of your k8s cluster (s3, external DB etc). You could have postgres running on the same k8s cluster as your other apps but that will be more of a Build-your-own solution which will have it drawbacks. 

## DSM UI

If you never used DSM the UI will give you a great overview. Lets provision a highly available PostgreSQL Cluster. 




DSM will now provision three VMs via vCenter and setup PostgreSQL within those three VMs (note: it will leverage kubernetes for that.

## DSM K8s Operator

Now we will provision a PostgreSQL Cluster via K8s Operator thats running inside our Kubernetes Cluster. 
