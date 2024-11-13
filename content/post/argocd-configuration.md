---
title: "Argocd Configuration" # Title of the blog post.
date: 2024-09-13T23:48:15+01:00 # Date of post creation.
description: "Article description." # Description used for search engine.
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
  - Kubernetes
tags:
  - Kubernetes 
  - CI/CD
  - VMware Tanzu
  - VMware Tanzu Kubernetes Grid
# comment: false # Disable comment if false.
---

# Intro
ArgoCD is a declarative, GitOps continuous delivery tool for Kubernetes. It's designed to make application deployment and lifecycle management automated, auditable, and easy to understand.

Key features and concepts of ArgoCD include:

- GitOps Methodology: Argo CD uses Git repositories as the source of truth for defining the desired application state.
- Kubernetes-native: It's implemented as a Kubernetes controller that continuously monitors running applications and compares their current state against the desired state specified in Git.

This blog post explains the deployment of ArgoCD in a Tanzu Kubernetes Grid Cluster. There is also the option to provision ArgoCD as a supervisor Service.

# Overview

ArgoCD typically uses a centralized deployment model with one instance managing multiple Kubernetes clusters. ArgoCD  uses a push-based architecture where workloads are pushed from a centralized cluster to remote clusters. Best Practise is to deploy ArgoCD on a dedicated Infrastructure/Management Kubernetes Cluster.

# ArgoCD Deployment on TKGS Guest Cluster

Create a namespace for ArgoCD

```
kubectl create namespace argocd
```

Apply the Argo CD installation manifest

```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Expose ArgoCD via nginx Ingress (Example):


```

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  rules:
  - host: argocd.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port: 
              name: https
tls:
- hosts:
  - argocd.example.com
  secretName: argocd-secret

```

If you have AKO installed:

```

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    kubernetes.io/ingress.class: avi
    ako.vmware.com/enable-tls: "true"
    ako.vmware.com/ssl-passthrough: "true"
spec:
  rules:
  - host: argocd.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port: 
              number: 443
  tls:
  - hosts:
    - argocd.example.com
    secretName: argocd-secret

```

Or you could use a standard Layer 4 LoadBalancer:

```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

To get the IP of your LoadBalancer (to create DNS Records):

```
export ARGOCD_SERVER=$(kubectl get svc argocd-server -n argocd -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
```

Get the Initial Admin Password:

```
export ARGO_PWD=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)```
```

# ArgoCD Configuration