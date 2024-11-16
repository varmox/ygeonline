---
title: "[Guide] Tanzu Kubernetes Cluster Example Deployments" # Title of the blog post.
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

# Intro
This post gives a few examples of Tanzu Kubernetes Cluster Manifests and how to deploy them.

### Prerequiries

- Successfull Installation of vSphere with Tanzu Supervisor
- vSphere Namespace created
- Already existing VM Classes, Storages Policies and Tanzu Kubernetes Releases assigned to the Namespace.

## Login via kubectl

You will find the IP to your Kubernetes API on the Namespace Option "Link to CLI Tools"

```
kubectl vsphere login --server=10.40.80.20
```

```
kubectl config get-contexts
```

```
kubectl config use-context <>
```

## Get Parameters of your vSphere Namespace

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
## Cluster with 3 Control and 6 Worker Nodes

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
   - replicas: 6
     name: worker
     vmClass: best-effort-large  
     storageClass: workload-management-storage-policy

```
Save this file as a yaml file on your workstation.

## Apply Cluster Manifest

```
kubectl apply -f clusterspecs.yaml
```

```
kubectl get tanzukubernetescluster
```

Check the Status of the Cluster. "READY" should be true after some minutes.

## Edge Cluster

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
    vmClass: best-effort-small  
    storageClass: workload-management-storage-policy
    tkr:
      reference:
       name: v1.23.8---vmware.2-tkg.2-zshippable
   nodePools:
   - replicas: 3
     name: worker
     vmClass: best-effort-medium  
     storageClass: workload-management-storage-policy

```

## Minimal Cluser for PoC
Use this cluster  only  for PoC/testing. Only three VMs are created in total.
Later, you'll use Kubernetes Taints to enable the Control node to  run user workloads.
This is useful when testing workloads with a minimal VM footprint when deploying in environments with few resources available.

> Concept of Kubernetes Taints
In Kubernetes, taints are a way to mark a node with a special attribute that affects which pods can be scheduled onto that node. Taints are used to repel or attract pods based on certain criteria, such as node characteristics or hardware specifications.


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
    vmClass: best-effort-medium  
    storageClass: workload-management-storage-policy
    tkr:
      reference:
       name: v1.23.8---vmware.2-tkg.2-zshippable
   
```
Do not use this in a production environment
After a successfull deployment of the cluster run the following commands to allow the Control Nodes to run User Workloads:

```
kubectl taint nodes --all node-role.kubernetes.io/master-
```