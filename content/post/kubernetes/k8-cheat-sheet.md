---
title: "K8 Cheat Sheet" # Title of the blog post.
date: 2022-12-04T23:16:50+01:00 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false  # Sets whether to render this page. Draft of true will not be rendered.
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

### kubectl Client Side
| kubectl command | description | Comment |
| --- | --- | --- |
| kubectl config view | View Kubectl Configuration | Display the current Kubectl configuration |
| kubectl config get-contexts | List available contexts | See the available Kubernetes clusters and their associated contexts |
| kubectl config current-context | Display the current context | Show the active Kubernetes cluster and context |
| kubectl config use-context <context_name> | Switch to another context | Change the active context to the specified one |
| kubectl config set-context <context_name> --namespace=<namespace_name> | Set default namespace for a context | Specify the default namespace for a specific context |
| kubectl config unset current-context | Unset the current context | Remove the association with the current context |
| kubectl config set-cluster <cluster_name> --server=<api_server_url> | Add a new cluster to the configuration | Define a new Kubernetes cluster with its API server URL |
| kubectl config set-credentials <user_name> --token=<access_token> | Add user credentials to the configuration | Specify user credentials, typically an access token |
| kubectl config set-context <context_name> --cluster=<cluster_name> --user=<user_name> | Create a new context | Combine a cluster and user to create a context |
| kubectl config delete-context <context_name> | Delete a context | Remove a specific context from the configuration |
| kubectl config delete-cluster <cluster_name> | Delete a cluster | Remove a cluster from the configuration |
| kubectl config delete-user <user_name> | Delete user credentials | Remove user credentials from the configuration |
| kubectl config use-context <context_name> | Switch to another context | Change the active context to the specified one |
| kubectl config rename-context <old_name> <new_name> | Rename a context | Change the name of an existing context |
| kubectl config set preferences.colors true | Enable colorized output | Enhance readability with colorized command output |


	
	
	

### Troubleshooting

| kubectl command | description | Comment |
| --- | --- | --- |
| kubectl get events | Check events for resources | View events for debugging and troubleshooting |
| kubectl describe node <node_name> | Run diagnostics on a specific node | Replace `<node_name>` with the actual node name |
| kubectl get componentstatuses | Check the health of cluster components | Verify the status of essential components in the cluster |
| kubectl get pods --all-namespaces | List all pods in all namespaces | Useful for identifying pods in any namespace |
| kubectl describe pod <pod_name> | Get detailed information about a specific pod | Replace `<pod_name>` with the actual pod name |
| kubectl logs <pod_name> | Retrieve the logs from a specific pod | Replace `<pod_name>` with the actual pod name |
| kubectl top nodes | Display resource usage statistics for nodes | View CPU and memory usage for all nodes |
| kubectl top pods | Display resource usage statistics for pods | View CPU and memory usage for all pods |

### Configuration

### Operations

 Run kubectl as non-root user \
`non-root@cp: ̃$ mkdir -p $HOME/.kube` \
 `non-root@cp: ̃$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config` \
 `non-root@cp: ̃$ sudo chown $(id -u):$(id -g) $HOME/.kube/config` \
 `non-root@cp: ̃$ less .kube/config` 