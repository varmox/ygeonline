---
title: "Tanzu Package Troubleshooting" # Title of the blog post.
postType: "QuickTip"
date: 2025-01-06T15:26:46+01:00 # Date of post creation.
description: "Common Error with Tanzu Packages" # Description used for search engine.
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
  - Kubernetes
tags:
  - Tanzu Packages
  - Troubleshooting
# comment: false # Disable comment if false.
---

# Cannot delete Tanzu Package

> #### **Error Messaage**
> Error: *Getting service account: serviceaccounts "grafana-tanzu-system-dashboards-sa" not found*

**Resolution**

```
kubectl get apps grafana -oyaml | grep finalizer
kubectl patch app grafana -n < my namespace > -p '{"metadata":{"finalizers":[]}}' --type=merge
kubectl delete app grafana
```
