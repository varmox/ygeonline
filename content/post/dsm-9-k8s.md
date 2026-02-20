---
title: "Data Services Manager - K8s " # Title of the blog post.
date: 2026-02-09T17:40:28+01:00 # Date of post creation.
description: "DBaaS with DSM and consume via Kubernetes" # Description used for search engine.
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
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - VMware
  - DSM
tags:
  - DSM
  - Data Services Manager
  - PostgreSQL
# comment: false # Disable comment if false.
---
# VMware Data Services Manager - K8s Operator

VMware DSM provisions Data Services like Postgres, MySQL and MSSQL. You can consume DSM in three ways:

- DSM UI 
- VCF Automation
- K8s Operator

DSM Installation is straightforward, just grab the OVA and install it. DSM will install a Plugin in vCenter and the DSM UI will be accessible with the IP you configured on the OVA setup. Nothing special needed, no NSX, no vSAN.

<img src="/images/dsm5.png" width="1100" alt="vExpert Badge" title="VMware Data Services Manager" />

## Scenario 1 - Postgres HA Cluster

We have the scenario that we have a Kubernetes cluster and some apps need a PostgreSQL database. In general, it is a good idea to have your persistent data outside of your k8s cluster (S3, external DB, etc.). You could have Postgres running on the same k8s cluster as your other apps, but that will be more of a build-your-own solution which will have its drawbacks.

### DSM UI

If you never used DSM the UI will give you a great overview. Lets provision a highly available PostgreSQL Cluster. 

<img src="/images/dsm1.png" width="1100" alt="vExpert Badge" title="DSM UI - Postgres Information" />

Select the Topology (Cluster = 3 VMs)

<img src="/images/dsm2.png" width="1100" alt="vExpert Badge" title="DSM UI - Topology" />


Select the Infrastructure Policy (vSphere Cluster, Portgroup, Storage Policy)

<img src="/images/dsm3.png" width="1100" alt="vExpert Badge" title="DSM UI - Infrastructure Policy" />

Configure Maintenance Windows and optional pg_hba parameters.

<img src="/images/dsm4.png" width="1100" alt="vExpert Badge" title="DSM UI - Additional Settings" />


DSM will now provision three VMs via vCenter and set up PostgreSQL within those three VMs (note: it will leverage Kubernetes for that).

### DSM K8s Operator

Now we will provision a PostgreSQL Cluster via K8s Operator thats running inside our Kubernetes Cluster. 

#### Install K8s Consumption Operator - DSM 2.x

Login to vSphere Kubernetes Cluster (VKS)

```bash
kubectl vsphere login --server=https://172.29.30.2 --insecure-skip-tls-verify --tanzu-kubernetes-cluster-name cl-lab-vbuenzli-kcl-01 --tanzu-kubernetes-cluster-namespace cl-lab-vbuenzli --vsphere-username "vbuenzli@sddc.lab"
```

create k8s namespace for dsm operator

```bash
kubectl create namespace dsm-consumption-operator-system
``` 

We are directly using the default registry, such as Broadcom Artifactory, you don't need authentication. We create a registry secret using the following command, ignoring the username and password values:

```bash
kubectl -n dsm-consumption-operator-system create secret docker-registry registry-creds \
  --docker-server=ignore \
  --docker-username=ignore \
  --docker-password=ignore
```

If you are using your own internal registry, where the consumption operator image exists, you need to provide those credentials here:

```bash
  kubectl -n dsm-consumption-operator-system create secret docker-registry registry-creds \
  --docker-server=<DOCKER_REGISTRY> \
  --docker-username=<REGISTRY_USERNAME> \
  --docker-password=<REGISTRY_PASSWORD>
```

Create an authentication secret that includes all the information needed to connect to the VMware Data Services Manager provider:

```bash
kubectl -n dsm-consumption-operator-system create secret generic dsm-auth-creds \
 --from-file=root_ca=dsm-root-ca \
 --from-literal=dsm_user=admin@dsm \
 --from-literal=dsm_password='VMware1!' \
 --from-literal=dsm_endpoint=https://172.29.30.51
```


Pull the helm chart from registry and unpack it in a directory and deploy to cluster.

Add your Backup Locations and Infrastructure Policies to the `values-override.yaml`. Infrastructure and Backup Locations can be created and viewed in the DSM UI.

values-override.yaml

```yaml
imagePullSecret: registry-creds
replicas: 1
image:
  name: projects.packages.broadcom.com/dsm-consumption-operator/consumption-operator
  tag: 2.2.0
dsm:
  authSecretName: dsm-auth-creds

  # allowedInfrastructurePolicies is a mandatory field that needs to be filled with allowed infrastructure policies for the given consumption cluster
  allowedInfrastructurePolicies:
  - labvce3-dsm-ip01

  # allowedBackupLocations is a mandatory field that holds a list of backup locations that can be used by database clusters created in this consumption cluster
  allowedBackupLocations:
  - vsan-datastore
  - artesca-lab
 
  # list of infraPolicies and backupLocations that need to be applied to all namespaces that match the given selector
  applyToNamespaces:
    selector:
      matchAnnotations: {}
    infrastructurePolicies: []
    backupLocations: []
    
   # adminNamespace specifies where administrative DSM system objects
   # should be stored in the consumption cluster. When not set, administrative
   # objects will not be available in the consumption cluster
  adminNamespace: "dsm-consumption-operator-system"
 
  # consumptionClusterName is an optional name that you can provide to identify the Kubernetes cluster where the operator is deployed
  consumptionClusterName: "infra01"

  # psp field allows you to deploy the operator on pod security policies-enabled Kubernetes cluster (ONLY for k8s version < 1.25).
  # Set psp.required to true and provide the ClusterRole corresponding to the restricted policy.
  psp:
    required: false
    role: ""
```

```bash
helm pull oci://projects.packages.broadcom.com/dsm-consumption-operator/dsm-consumption-operator --version 2.2.0 -d consumption/ --untar

helm upgrade --install dsm-consumption-operator consumption/dsm-consumption-operator -f values-override.yaml --namespace dsm-consumption-operator-system
```

Check Logs:

```bash
kubectl logs  -n dsm-consumption-operator-system deployments/dsm-consumption-operator-controller-manager -f
```

#### Install K8s Consumption Operator - DSM 9.0.1

In DSM 9.0.1, two things have changed regarding the k8s operator. See the Vendor Docs [2]


- For VMware Data Services Manager version 9.0.1 and later, add a dataServicePolicyNamespaceLabels field to the values.yaml file.
- For VMware Data Services Manager version 9.0.1 and later, you create data service policies in the VMware Data Services Manager gateway.

## Create PostgreSQL Cluster via K8s Consumption Operator

Create PostgresCluster Manifest:
```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: dev-team
---
apiVersion: infrastructure.dataservices.vmware.com/v1alpha1
kind: InfrastructurePolicyBinding
metadata:
  name: labvce3-dsm-ip01
  namespace: dev-team
---
apiVersion: databases.dataservices.vmware.com/v1alpha1
kind: BackupLocationBinding
metadata:
  name: artesca-lab
  namespace: dev-team
---
apiVersion: databases.dataservices.vmware.com/v1alpha1
kind: PostgresCluster
metadata:
  name: pg-dev-cluster
  namespace: dev-team
spec:
  replicas: 3
  version: "14"
  vmClass:
    name: medium
  storageSpace: 20Gi
  infrastructurePolicy:
    name: labvce3-dsm-ip01
  storagePolicyName: sp-dsm-e3
  backupConfig:
    backupRetentionDays: 91
    schedules:
      - name: full-weekly
        type: full
        schedule: "0 0 * * 0"
      - name: incremental-daily
        type: incremental
        schedule: "30 10 * * *"
  backupLocation:
    name: artesca-lab
```

Apply it:

```bash
kubectl apply -f user-namespace-example.yaml 

export K8S_NAMESPACE=dev-team
```

> View available infrastructure policies by running the following command:

```bash
kubectl get infrastructurepolicybinding -n $K8S_NAMESPACE
``` 
You can also check the status field of each infrapolicybinding to find out the values of vmClass, storagePolicy, and so on.


Check status of ongoing deployment:

```bash
kubectl get postgresclusters.databases.dataservices.vmware.com -n $K8S_NAMESPACE pg-dev-cluster -o jsonpath='{.status.conditions}' | jq
```
Wait for:

```json
    {
      "lastTransitionTime": "2025-06-26T06:24:40Z",
      "message": "",
      "observedGeneration": 1,
      "reason": "ConfigApplied",
      "status": "True",
      "type": "CustomConfigStatus"
    }
```

Retrieve Connection Information

```bash
kubectl get postgresclusters.databases.dataservices.vmware.com -n $K8S_NAMESPACE pg-dev-cluster -o jsonpath='{.status.connection}' | jq
```

```json
{
  "dbname": "pg-dev-cluster",
  "host": "10.177.150.86",
  "passwordRef": {
    "name": "pg-pg-dev-cluster"
  },
  "port": 5432,
  "username": "pgadmin"
}
```

Retrieve Connection Vars:

```bash
PASSWORD=$(kubectl -n $K8S_NAMESPACE get secrets/$(kubectl get postgresclusters.databases.dataservices.vmware.com -n $K8S_NAMESPACE pg-dev-cluster -o jsonpath='{.status.connection.passwordRef.name}') --template={{.data.password}} | base64 -d)
USER=$(kubectl get postgresclusters.databases.dataservices.vmware.com -n $K8S_NAMESPACE pg-dev-cluster -o jsonpath='{.status.connection.username}')
HOST=$(kubectl get postgresclusters.databases.dataservices.vmware.com -n $K8S_NAMESPACE pg-dev-cluster -o jsonpath='{.status.connection.host}')
DBNAME=$(kubectl get postgresclusters.databases.dataservices.vmware.com -n $K8S_NAMESPACE pg-dev-cluster -o jsonpath='{.status.connection.dbname}')
``` 

Connect to the DB:

```bash
PGPASSWORD=$PASSWORD psql -h $HOST -U $USER $DBNAME
```


### Test Deployment

#### Patch existing secret

```bash
# Define variables
K8S_NAMESPACE="dev-team"
CLUSTER_NAME="pg-dev-cluster"
SECRET_NAME=$(kubectl get postgresclusters.databases.dataservices.vmware.com -n $K8S_NAMESPACE $CLUSTER_NAME -o jsonpath='{.status.connection.passwordRef.name}') # The secret you want to patch

# Fetch the connection info as a JSON object
CONNECTION_INFO=$(kubectl get postgresclusters.databases.dataservices.vmware.com \
  -n "$K8S_NAMESPACE" "$CLUSTER_NAME" \
  -o jsonpath='{.status.connection}')

# Check if the command succeeded
if [ -z "$CONNECTION_INFO" ]; then
  echo "Error: Could not retrieve connection info for cluster '$CLUSTER_NAME' in namespace '$K8S_NAMESPACE'."
  exit 1
fi

# Extract values using jq
HOST=$(echo "$CONNECTION_INFO" | jq -r '.host')
PORT=$(echo "$CONNECTION_INFO" | jq -r '.port')
DBNAME=$(echo "$CONNECTION_INFO" | jq -r '.dbname')
USERNAME=$(echo "$CONNECTION_INFO" | jq -r '.username')

# Construct the JSON patch payload using stringData
PATCH_PAYLOAD=$(cat <<EOF
{
  "stringData": {
    "host": "$HOST",
    "port": "$PORT",
    "dbname": "$DBNAME",
    "username": "$USERNAME"
  }
}
EOF
)



# Apply the patch to the secret
echo "Patching secret '$SECRET_NAME' in namespace '$K8S_NAMESPACE'..."
kubectl patch secret "$SECRET_NAME" -n "$K8S_NAMESPACE" --type=merge -p "$PATCH_PAYLOAD"



printf "Patch complete. \n Secret: \n $(kubectl get secret "$SECRET_NAME" -n "$K8S_NAMESPACE" -o yaml ) \n"

# create new secret

kubectl create secret generic "pg-demo" --from-literal=host=$HOST --from-literal=port=$PORT --from-literal=dbname=$DBNAME --from-literal=username=$USERNAME -n "$K8S_NAMESPACE"  --from-literal=password=$PASSWORD
```

## Test Application

Retrieve Connection Vars:

```bash
export PG_PASSWORD=$(kubectl -n $K8S_NAMESPACE get secrets/$(kubectl get postgresclusters.databases.dataservices.vmware.com -n $K8S_NAMESPACE pg-dev-cluster -o jsonpath='{.status.connection.passwordRef.name}') --template={{.data.password}} | base64 -d)
export PG_USER=$(kubectl get postgresclusters.databases.dataservices.vmware.com -n $K8S_NAMESPACE pg-dev-cluster -o jsonpath='{.status.connection.username}')
export PG_HOST=$(kubectl get postgresclusters.databases.dataservices.vmware.com -n $K8S_NAMESPACE pg-dev-cluster -o jsonpath='{.status.connection.host}')
export PG_DBNAME=$(kubectl get postgresclusters.databases.dataservices.vmware.com -n $K8S_NAMESPACE pg-dev-cluster -o jsonpath='{.status.connection.dbname}')
``` 

Label the namespace to allow workload without matching security context:

```bash
kubectl label namespace $K8S_NAMESPACE \
  pod-security.kubernetes.io/audit=privileged \
  pod-security.kubernetes.io/warn=privileged \
  pod-security.kubernetes.io/enforce=privileged \
  --overwrite
```  


Test App Manifest (dsm-test-deployment.yaml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pgtestapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pgtestapp
  template:
    metadata:
      labels:
        app: pgtestapp
    spec:
      imagePullSecrets:
      - name: regcred
      containers:
      - name: pgtestapp
        image: ghcr.io/kastenhq/pgtest:v0.0.1
        imagePullPolicy: Always
        # command: 
        #   - "/bin/sh"
        #   - "-c"
        #   - |
        #     echo "PG_HOST: $PG_HOST"
        #     echo "PG_DBNAME: $PG_DBNAME"
        #     echo "PG_USER: $PG_USER"    
        #     echo "PG_PASSWORD: $PG_PASSWORD"
        #     tail -f /dev/null
        ports:
        - containerPort: 8080
        env:
          - name: PG_HOST
            valueFrom:
              secretKeyRef:
                name: pg-pg-dev-cluster
                key: host
          - name: PG_DBNAME
            valueFrom:
              secretKeyRef:
                name: pg-pg-dev-cluster
                key: dbname
          - name: PG_USER
            valueFrom:
              secretKeyRef:
                name: pg-pg-dev-cluster
                key: username
          - name: PG_PASSWORD
            valueFrom:
              secretKeyRef:
                name: pg-pg-dev-cluster
                key: password
---
apiVersion: v1
data:
  .dockerconfigjson: eyJhdXRocyI6eyJnaGNyLmlvIjp7InVzZXJuYW1lIjoidHJpYm9jayIsInBhc3N3b3JkIjoiZ2hwX1hpVmYxZFVGYUdCVDRzeHo5NHRUZTVDeGNIRjh5djNMUG5YOSIsImVtYWlsIjoiY2ljZEBzb3VsdGVjLmNoIiwiYXV0aCI6ImRISnBZbTlqYXpwbmFIQmZXR2xXWmpGa1ZVWmhSMEpVTkhONGVqazBkRlJsTlVONFkwaEdPSGwyTTB4UWJsZzUifX19
kind: Secret
metadata:
  creationTimestamp: null
  name: regcred
type: kubernetes.io/dockerconfigjson
---
apiVersion: v1
kind: Service
metadata:
  name: pgtestapp
spec:
  ports:
    - port: 8080
  selector:
    app: pgtestapp
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pg-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pg-demo
  template:
    metadata:
      labels:
        app: pg-demo
    spec:
      imagePullSecrets:
      - name: regcred
      containers:
      - name: pgtestapp
        image: ghcr.io/kastenhq/pgtest:v0.0.1
        imagePullPolicy: Always
        # command: 
        #   - "/bin/sh"
        #   - "-c"
        #   - |
        #     echo "PG_HOST: $PG_HOST"
        #     echo "PG_DBNAME: $PG_DBNAME"
        #     echo "PG_USER: $PG_USER"    
        #     echo "PG_PASSWORD: $PG_PASSWORD"
        #     tail -f /dev/null
        ports:
        - containerPort: 8080
        env:
          - name: PG_HOST
            valueFrom:
              secretKeyRef:
                name: pg-demo
                key: host
          - name: PG_DBNAME
            valueFrom:
              secretKeyRef:
                name: pg-demo
                key: dbname
          - name: PG_USER
            valueFrom:
              secretKeyRef:
                name: pg-demo
                key: username
          - name: PG_PASSWORD
            valueFrom:
              secretKeyRef:
                name: pg-demo
                key: password    

```  

Deploy the Application:

```bash
kubectl apply  -f dsm-test-deployment.yaml -n $K8S_NAMESPACE 

```

> Docker Image used for Testing: https://github.com/kastenhq/pgtest


# Summary

Now we have successfully deployed a PostgreSQL Cluster with the DSM K8s Operator (from our Kubernetes Guest Cluster) and deployed a sample application that used the dsm-provisioned database.

In the next blog we are going to have a look at the third option on how to consume db's via DSM, VCF Automation (VCFA)

<img src="/images/dsm-vcfa.png" width="1100" alt="vExpert Badge" title="VCF Automation - Data Services" />


# Resources

[1] [DSM Intro - VMware Blog from my friend Thomas](https://blogs.vmware.com/cloud-foundation/2024/09/24/the-value-of-data-services-manager-for-modern-workloads/)

[2]  [DSM K8s Operator - Vendor Doc](https://techdocs.broadcom.com/us/en/vmware-cis/dsm/data-services-manager/9-0/working-with-databases-in-vmware-data-services-manager/enabling-self-service-consumption-of-vmware-data-services-manager/step-2-install-the-dsm-consumption-operator.html)

[3] [DSM MSSQL - First Look](https://soultec.ch/dsm-9-0-mssql-support/)
