---
title: "K8 Cheat Sheet" # Title of the blog post.
date: 2022-12-04T23:16:50+01:00 # Date of post creation.
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
  - Kubernetes
tags:
  - Cheat Sheet
  - Kubernetes
# comment: false # Disable comment if false.
---

**Kubernetes Cheat Sheet**

Cheat Sheet for kubectl, kubeadm and general k8s related commands.

### Troubleshooting

### Configuration

### Operations

 Run kubectl as non-root user \
`non-root@cp: ̃$ mkdir -p $HOME/.kube` \
 `non-root@cp: ̃$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config` \
 `non-root@cp: ̃$ sudo chown $(id -u):$(id -g) $HOME/.kube/config` \
 `non-root@cp: ̃$ less .kube/config` 