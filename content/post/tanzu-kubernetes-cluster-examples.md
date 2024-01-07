---
title: "Tanzu Kubernetes Cluster Examples" # Title of the blog post.
date: 2023-06-23T23:13:14+02:00 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
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
  - VMware Tanzu
  - Tanzu Kubernetes Clusters
  - yaml
# comment: false # Disable comment if false.
---

**Cluster with 3 Master and 3 Worker Nodes**

```
apiVersion: run.tanzu.vmware.com/v1alpha3
kind: TanzuKubernetesCluster
metadata:
 name: tkgs-cluster
 namespace: tkgs-cluster-ns   
spec:
  topology:
   controlPlane:
    replicas: 3
    vmClass: best-effort-large  
    storageClass: workload-management-storage-policy
    tkr:
      reference:
       name: v1.23.8---vmware.2-tkg.2-zshippable
   nodePools:
   - replicas: 3
     name: worker
     vmClass: best-effort-large  
     storageClass: workload-management-storage-policy

```
Save this file as a yaml file on your workstation.


**Get Parameters of your vSphere Namespace**

- VM Class
- Storagge Policy
- Tanzu Releases

```
kubectl get vmclass
```

```
kubectl get storageclass
```

```
kubectl get tanzukubernetesrelease
```

**Apply Cluster Manifest**

```
kubectl apply -f clusterspecs.yaml
```

```
kubectl get tanzukubernetescluster
```