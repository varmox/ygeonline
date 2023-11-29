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

### Nodes
| kubectl command | description | Comment |
| --- | --- | --- |
| kubectl get nodes | List all nodes in the cluster | Display a list of nodes and their status |
| kubectl describe node <node_name> | Get detailed information about a node | Replace `<node_name>` with the actual node name |
| kubectl get nodes -o wide | Display additional details about nodes | View IP addresses and other information about nodes |
| kubectl get nodes --show-labels | Show labels assigned to nodes | Display labels associated with each node |
| kubectl get nodes -o json | Display node information in JSON format | View detailed node information in JSON |
| kubectl cordon <node_name> | Mark a node as unschedulable | Prevent new pods from being scheduled on the specified node |
| kubectl uncordon <node_name> | Mark a node as schedulable | Allow new pods to be scheduled on the specified node |
| kubectl drain <node_name> | Safely evict all pods from a node | Evacuate a node for maintenance |
| kubectl top nodes | Display resource usage statistics for nodes | View CPU and memory usage for all nodes |
| kubectl get nodes --sort-by=.status.capacity.cpu | Sort nodes by CPU capacity | Sort nodes based on CPU capacity in ascending order |
| kubectl label nodes <node_name> <label_key>=<label_value> | Label a node | Assign a label to a specific node |
| kubectl taint nodes <node_name> <taint_key>=<taint_value>:<effect> | Taint a node | Apply a taint to a node for node affinity or tolerations |
| kubectl describe nodes | Get detailed information about all nodes | View detailed information about all nodes in the cluster |
| kubectl get componentstatuses | Check the health of cluster components | Verify the status of essential components in the cluster |
| kubectl get nodes -o custom-columns=NAME:.metadata.name,STATUS:.status.phase | Display custom columns for nodes | Define custom output columns for nodes |

### Pods
| kubectl command | description | Comment |
| --- | --- | --- |
| kubectl get pods | List all pods in the current namespace | Display a list of pods and their status |
| kubectl get pods -n <namespace> | List pods in a specific namespace | Replace `<namespace>` with the desired namespace |
| kubectl describe pod <pod_name> | Get detailed information about a pod | Replace `<pod_name>` with the actual pod name |
| kubectl logs <pod_name> | Retrieve the logs from a specific pod | Replace `<pod_name>` with the actual pod name |
| kubectl exec -it <pod_name> -- <command> | Execute a command in a running pod | Replace `<pod_name>` with the actual pod name and `<command>` with the desired command |
| kubectl port-forward <pod_name> <local_port>:<pod_port> | Forward a pod's port to the local machine | Replace `<pod_name>`, `<local_port>`, and `<pod_port>` with the actual values |
| kubectl delete pod <pod_name> | Delete a specific pod | Replace `<pod_name>` with the actual pod name |
| kubectl delete pods --all | Delete all pods in the current namespace | Be cautious when using this command |
| kubectl apply -f <pod_manifest.yaml> | Create or update a pod using a manifest file | Replace `<pod_manifest.yaml>` with the path to the pod's YAML manifest |
| kubectl get pod <pod_name> -o yaml | Get the YAML definition of a pod | Replace `<pod_name>` with the actual pod name |
| kubectl exec -it <pod_name> -- /bin/bash | Open a shell in a running pod for debugging | Replace `<pod_name>` with the actual pod name |
| kubectl describe pod <pod_name> \| grep Image | Get the container image version running in a pod | Replace `<pod_name>` with the actual pod name |


### Storage
| kubectl command | description | Comment |
| --- | --- | --- |
| kubectl get storageclass | List available storage classes | Check the available storage classes in the cluster |
| kubectl describe storageclass <storageclass_name> | Get detailed information about a storage class | Replace `<storageclass_name>` with the actual storage class name |
| kubectl get persistentvolumes | Display information about persistent volumes | View details of persistent volumes in the cluster |
| kubectl describe persistentvolume <pv_name> | Get detailed information about a persistent volume | Replace `<pv_name>` with the actual persistent volume name |
| kubectl get persistentvolumeclaims | List persistent volume claims | View information about persistent volume claims in the current namespace |
| kubectl describe persistentvolumeclaim <pvc_name> | Get detailed information about a persistent volume claim | Replace `<pvc_name>` with the actual persistent volume claim name |
| kubectl get pv,pvc | List both persistent volumes and claims | Display a combined list of persistent volumes and claims |
| kubectl apply -f <storage_manifest.yaml> | Create or update storage using a manifest file | Replace `<storage_manifest.yaml>` with the path to the storage YAML manifest |
| kubectl delete persistentvolume <pv_name> | Delete a specific persistent volume | Replace `<pv_name>` with the actual persistent volume name |
| kubectl delete persistentvolumeclaims --all | Delete all persistent volume claims in the current namespace | Be cautious when using this command |
| kubectl get storageclass -o yaml | Get the YAML definition of a storage class | Replace `<storageclass_name>` with the actual storage class name |
| kubectl get persistentvolume -o yaml | Get the YAML definition of a persistent volume | Replace `<pv_name>` with the actual persistent volume name |
| kubectl get persistentvolumeclaim -o yaml | Get the YAML definition of a persistent volume claim | Replace `<pvc_name>` with the actual persistent volume claim name |
| kubectl exec -it <pod_name> -- df -h | Check storage usage inside a pod | Replace `<pod_name>` with the actual pod name and `df -h` with the desired command |
| kubectl describe storageclass <storageclass_name> \| grep Provisioner | Get the provisioner information for a storage class | Replace `<storageclass_name>` with the actual storage class name |


### Operations

 Run kubectl as non-root user \
`non-root@cp: ̃$ mkdir -p $HOME/.kube` \
 `non-root@cp: ̃$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config` \
 `non-root@cp: ̃$ sudo chown $(id -u):$(id -g) $HOME/.kube/config` \
 `non-root@cp: ̃$ less .kube/config` 