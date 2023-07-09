---
title: "Huawei VRP" # Title of the blog post.
date: 2022-12-04T15:11:13+01:00 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
usePageBundles: false # Set to true to group assets like images in the same folder as this post.
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
featureImageAlt: 'Description of image' # Alternative text for featured image.
featureImageCap: 'This is the featured image.' # Caption (optional).
#thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Technology
tags:
  - Networking
  - Huawei
# comment: false # Disable comment if false.
---

**Huawei Versatile Routing Platform (VRP) Overview**

This post gives an overview about the network operating system for huawei network devices - VRP.


**General**

|  Task |  Command | Comment  |  
|---|---|---|
| Enter System View   |  < Huawei > sys  
| Enter Interface View   |  [R1] int GigabitEthernet0/0/1  |   |   |   |
| Enter Protocol View  |  [R1] ospf 1  |   |   |   |
 

 

 

 

 

 







 





[R1-GigabitEthernet0/0/1] 

 





 

 

Display 

 

 

Display Config in Current View 

[R1- GigabitEthernet0/0/3]display this 

# interface GigabitEthernet0/0/3 ip address 10.0.12.1 255.255.255.0 ospf authentication-mode md5 1 cipher foCQTYsq-4.A\^38y!DVwQ0# 

# 

 

Display current configuration of the System 

[R1]dis current-config 

 

 

 

 

General 

 

  

Set hostname 

[Huawei]sysname R1 

[R1] 

 

 

 

 

