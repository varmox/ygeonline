---
title: "Cillium Gateway API - Getting Started" # Title of the blog post.
date: 2023-10-14T18:18:26+02:00 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: true # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
usePageBundles: false # Set to true to group assets like images in the same folder as this post.
featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
featureImageAlt: 'Description of image' # Alternative text for featured image.
featureImageCap: 'This is the featured image.' # Caption (optional).
thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Technology
tags:
  - Kubernetes Cillium
  - CNI
# comment: false # Disable comment if false.
---

**Cillium Gateway API**
The Cilium Gateway API is a Kubernetes API that provides a more powerful and flexible way to manage traffic routing than the traditional Ingress API of (vanilla) Kubernetes. It is a set of resources that model service networking in Kubernetes, and is designed to be role-oriented, portable, expressive, and extensible.
The Cilium Service Mesh Gateway API Controller requires the ability to create LoadBalancer Kubernetes services.