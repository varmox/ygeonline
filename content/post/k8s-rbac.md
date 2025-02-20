---
title: "[Guide] RBAC on VKS" # Title of the blog post.
date: 2024-12-08T12:29:05+01:00 # Date of post creation.
description: "RBAC on vSphere Kubernetes Service" # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: true # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
usePageBundles: false # Set to true to group assets like images in the same folder as this post.
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
#featureImageAlt: 'Description of image' # Alternative text for featured image.
#featureImageCap: 'This is the featured image.' # Caption (optional).
#thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 80 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Kubernetes
  - VMware
tags:
  - vSphere Kubernetes Service
  - Tanzu Kubernetes
  - vSphere with Tanzu
  - Tanzu Kubernetes Grid
  
# comment: false # Disable comment if false.
---

# Intro

This Blogpost explains RBAC on vSphere Kubernetes Service (VKS), formely known as Tanzu Kubernetes Grid Service (TKGS). It also shows how to give access via *kubectl* with granular permissions, leveraging OIDC Auth with pinniped on VKS-Clusters.

> #### **Naming**
> I use VKS-, TKGS- and K8s Clusters in this Blog Post. All have the same meaning - simply a Kubernetes Cluster :)


# RBAC on vSphere Kubernetes Service

Role-based Access Control on vSphere Kubernetes Cluster has a few Keypoint which should be though about. The main question you have to ask yourself: Which Level of access do I want to grant to which people?

## Option 1: vSphere Namespace Permissions

With vSphere Namespace Permissions ([Select Permissions > Add Permissions](https://docs.vmware.com/en/VMware-vSphere/8.0/vsphere-with-tanzu-tkg/GUID-223D91FB-C4CB-4DA7-8B3F-24721ABDFBC7.html)) you grant access directly on the vSphere Namespace via vCenter.

![vSphere Namespace Permissions](https://i.imgur.com/k3ARB75.png)


| Option   | Description                                                                                                                                  |
|----------|----------------------------------------------------------------------------------------------------------------------------------------------|
| Can View | Can read TKG cluster objects in the vSphere Namespace. No permissions mapped to Kubernetes roles. See Role Permissions and Bindings.         |
| Can Edit | Can create, read, update, and delete TKG cluster objects in the vSphere Namespace. Can operate TKG clusters provisioned in the vSphere Namespace as the Kubernetes cluster-admin. See Role Permissions and Bindings. |
| Owner    | Same permissions as Can Edit, with the additional permission to create and manage vSphere Namespaces using kubectl. Only available with vCenter SSO. See Role Permissions and Bindings. |



This means - depending on the Permission you give:


- User has Access to all Ressources within that vSphere Namespace:
  - all TKGS/VKS Guest Clusers (kubectl)
  - all VMs wihtin that Namespace
- Create new TKGS/VKS Clusters
- Create new VMs

But I only want to give "some" permissions to the Kubernetes Cluster for my devs - now Kubernetes RBAC Concept comes into play.

## Option 2: K8s RBAC

With Kubernetes RBAC you have way more granular control over the permission:

**Core Components of Kubernetes RBAC**

- **Roles**: Define a set of permissions for accessing Kubernetes resources within a single namespace.
- **ClusterRoles**: Similar to Roles but cluster-scoped, allowing permissions across all namespaces and for cluster-wide resources.
- **RoleBindings**: Grant the permissions defined in a Role to users or service accounts within a specific namespace.
- **ClusterRoleBindings**: Cluster-scoped version of RoleBindings, granting permissions defined in a ClusterRole across the entire cluster.

```
                 +-------------+
                 |    User    |
                 +------+------+
                        |
                        v
              +-------------------+
              | Authentication    |
              +--------+----------+
                       |
                       v
              +-------------------+
              |   Authorization   |
              +--------+----------+
                       |
                       v
         +-------------+-------------+
         |                           |
         v                           v
  +-------------+            +----------------+
  |    Roles    |            |  ClusterRoles  |
  +------+------+            +--------+-------+
         |                            |
         |              +-------------+
         |              |
         v              v
  +-------------+  +-----------------+
  | RoleBinding |  | ClusterRoleBinding |
  +------+------+  +-----------------+
         |                   |
         |    +--------------+
         |    |
         v    v
    +----+----+----+
    |  Resources   |
    | (Namespaced) |
    +-------------+
```
1.) The process starts with a user attempting to access the cluster.

2.) Authentication verifies the user's identity.

3.) Authorization (RBAC) determines what actions the user can perform.

4.) Roles and ClusterRoles define permissions for resources.

5.) RoleBindings and ClusterRoleBindings associate users with roles.

6.) Finally, access is granted or denied to the requested resources based on the RBAC configuration.


## When to choose what

Choosing between vSphere Namespace Permissions and K8s RBAC can be simplified to the following:

- Full Access to the all VKS/TKGS Cluster within a vSphere Namespace -> vSphere Namespace Permissions
- Granular Access to VKS/TKGS Cluster or even just K8s Namespace -> K8s RBAC

## K8s RBAC on VKS with pinniped

**Scenario: Developer Access to a K8s Namespace**

Scenario Intro:

- Your companies Dev-Team  needs *kubectl* access to a K8s Namespace on a shared VKS/TKGS Cluster.
- You (Infra Admin) have vSphere Namespace Permissions of "Edit" or "Owner", also admin rights to the vSphere Supervisor
- You have a OIDC Provider (GitLab, WorkspaceONE Access, etc) already in place


We will configure the infrastructure that the following is possible:

- Devs Team will have a kubeconfig File, which they can use to logon via *kubectl*
- GitLab will act as a SSO Provider
- Role and RoleBindings are applied to the VKS-Cluster to grant permissions to the K8s-Namespace.

```
+------------------+
|      User        |
+--------+---------+
         |
         v
+------------------+    +--------------------------------------------------------+
| Identity Provider|<-->|Pinniped Supervisor - running on the vSphere Supervisor |
+------------------+    +--------------------------------------------------------+
                               ^
                               |
                        +------+------+
                        |   Pinniped  |
                        |  Concierge  |
                        +------+------+
                               |
                               v
+------------------+    +------------------+
|    vSphere       |    | Tanzu Kubernetes |
|    Namespace     |    |     Cluster      |
+------------------+    +------------------+
```


### Configure SSO with vSphere Supervisor

First we will configure the vSphere Supervisor to use GitLab as a Identity Provider. For that we need to create a OIDC Application in GitLab:

- Go to your GitLab Admin Panel (eg. *https://gitlab.yourdomain.tld/admin/applications* )
  - Create a Application
    - Example:
      - Name: my-supervisor.yourdomain.tld
      - Redirect URI: *https://supervisor-fqdn/wcp/pinniped/callback*
      - Scopes:
          - read_user
          - openid
          - profile
          - email


[Futher Information: GitLab Docs](https://docs.gitlab.com/ee/integration/openid_connect_provider.html)


Then configure GitLab as a Identity Provider in the vSphere Supervisor:

![vSphere Supervisor Identity Provider](https://i.imgur.com/rRTQ38N.png)

Example:

  - Provider Name: *gitlab-oidc*
  - Issuer URL: *https://gitlab.mydomain.local*
  - Username Claim: *nickname*
  - Groups Claim: *groups*
  - Client ID: *the ID from your GitLab Application*
  - Client Secret: *the Secret from your GitLab Application*
  - Additional Scopes: *openid*
  - Certififacte Authority Data: *PEM Formatted Cert*



The configuration of a Idendity Provider will install pinniped supervisor on the vSphere Namespace. Pinniped is used for OIDC.



### Role and RoleBinding

> #### **GitLab Configuration**
> Be careful with the "name: my-gitlab-group" within your Kubernetes Role. It really depends on how the GitLab Group was created. I > personally had problems with  nested groups or groups created on projects. I always created the GitLab Groups/Membership with a 
> Admin, didn't use the Group from a GitLab project.

Further we need a *Role* and a *RoleBinding*. Apply those on the actual Kubernetes Cluster.

**RoleBinding**
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rbac-default-developer-rolebinding
  namespace: NAMESPACE_NAME
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: rbac-default-developer-role
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: my-gitlab-group
```




**Role**

*Adjust the permissions as needed*

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: rbac-default-developer-role
  namespace: my-k8s-namespace
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["", "apps", "batch"]
  resources: ["pods", "services", "deployments", "replicasets", "statefulsets", "jobs", "cronjobs", "pods/exec"]
  verbs: ["create", "get", "list", "watch", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["configmaps", "secrets", "persistentvolumeclaims"]
  verbs: ["get", "list", "watch", "patch", "delete", "create","update"]
- apiGroups: ["networking.k8s.io"]
  resources: ["ingresses"]
  verbs: ["create", "get", "list", "watch", "update", "patch", "delete"]
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["cluster.x-k8s.io"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["namespaces"]
  verbs: ["get", "list", "watch"]
```




### Create the kubeconfig File

> #### **Permissions**
> Use your Account that has "Edit" or "Owner" Permissions on the vSphere Namespace, where the VKS-Cluster is residing in.

To create a kubeconfig file run the following commands:

```
tanzu context create my-kubernetes-cluster --endpoint https://SupervisorVIP
```

```
tanzu cluster kubeconfig get my-kubernetes-cluster -n my-vSphere-Namespace --export-file my-kubeconfig
```

**Note**: The kubeconfig File does not store any personalized credentials, so can safely be shared with your devs. The kubeconfig also isn't specific to a K8s-Namespace. The kubeconfig is specific to a Kubernetes Cluster (kubeapi-server IP).


You could also use the Local Consumption Interface to download the kubeconfig:

![Local Consumption Interface - LCI](https://i.imgur.com/rZ7yMXW.png)


### Test the Access

Either let the user test or you can use the following command with your administrator account on the k8s cluster:

```
kubectl auth can-i get pods -n namespace --as=user
```


**From a Dev's point of view, do the following:**

Install Tanzu CLI and the pinniped auth Plugin:

```
tanzu plugin install pinniped-auth
```

Run kubectl with a the created kubeconfig file.
```
kubectl --kubeconfig ./my-kubeconfig get ns
```

Now you will be given a link (to the pinniped Service running on the vSphere Supervisor). Copy this link into your browser, GitLab will open (if not already logged in) and authenticate you. Afterwards you will be presented with a Token. Copy this token back in to your terminal to get access to your kubernetes cluster or namespace.

```
$ kubectl --kubeconfig ./my-kubeconfig get pods

Optionally, paste your authorization code: G2TcS145Q4e6A1YKf743n3BJlfQAQ_UdjXy38TtEEIo.ju4QV3PTsUvOigVUtQllZ7AJFU0YnjuLHTRVoNxvdZc

✔ successfully logged in to management cluster using the kubeconfig 
Checking for required plugins...
All required plugins are already installed and up-to-date

NAME                           READY   STATUS    RESTARTS   AGE
nginx-deployment-66b6c48dd5-7tzxb   1/1     Running   0          3d
nginx-deployment-66b6c48dd5-8p4vk   1/1     Running   0          3d
nginx-deployment-66b6c48dd5-f8t5r   1/1     Running   0          3d
```

Optional: Overwrite or create the default kubeconfig file:
```
cp my-kubeconfig ~/.kube/config
```

Et voila, now your Developer has access via *kubectl*.


**Scenario: Developer Access to a K8s Cluster, but not will full permission**

Basically the same as above, but now you will use a K8s *ClusterRole* and a *ClusterRoleBinding* instead of a *Role* & *RoleBinding*.

**ClusterRole**

*Adjust the permissions as needed*

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: rbac-default-cluster-operator-clusterrole
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["namespaces", "configmaps", "pods", "secrets", "persistentvolumeclaims", "persistentvolumes", "services"]
  verbs: ["create", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets", "statefulsets"]
  verbs: ["create", "patch", "delete"]
- apiGroups: ["networking.k8s.io"]
  resources: ["ingressclasses", "ingresses","networkpolicies"]
  verbs: ["create", "patch", "delete"]
```

**ClusterRoleBinding**
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: rbac-default-cluster-operator-clusterrolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: rbac-default-cluster-operator-clusterrole
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: my-gitlab-group
  ```


## Further Thoughts

**GitOps your K8s RBAC!**

- Define a default ClusterRole (for K8s-Cluster-Access)
- Define a default Role (for K8s-Namespace Access)
- Use ArgoCD or FluxCD to roll-out your RBAC Config

***Token Lifetime**

Depending on your IdP, the Token TTL defines how many times the Dev have to re-authenticate.
GitLab isn't really configurabe as a "real" IdP. Look at WorkspaceONE Access, Zitadel for further configurations. 