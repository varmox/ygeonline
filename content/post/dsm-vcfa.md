---
title: "Data Services Manager with VCF Automation" # Title of the blog post.
postType: "Guide"
date: 2026-04-20T21:03:16+02:00 # Date of post creation.
description: "How to set up Data Services Manager with VCF Automation to provision PostgreSQL clusters."
featured: true
draft: true
toc: true
usePageBundles: false
codeMaxLines: 10
codeLineNumbers: false
figurePositionShow: true
categories:
  - VMware
  - VCFA
  - DSM
tags:
  - VMware
  - DSM
  - VCFA
# comment: false # Disable comment if false.
---

# Data Services Manager and VCF Automation

DSM resources can be provisioned through the DSM UI, Kubernetes Operator, and VCF Automation. This post covers the DSM and VCF Automation integration.

<img src="/images/dsm5.png" width="1100" alt="DSM Overview" title="VMware Data Services Manager" />

Our goal is to provision a PostgreSQL cluster through VCF Automation.

<img src="/images/vcfa-dsm/vcfa-1.png" width="1100" alt="VCFA DSM provisioning" title="VMware Data Services Manager" />

<img src="/images/vcfa-dsm/vcfa-2.png" width="1100" alt="VCFA DSM provisioning details" title="VMware Data Services Manager" />

# Prerequisites

VCFA leverages VKS / Supervisor for the DSM integration:

- Infrastructure Policies for DSM UI
- vSphere Namespaces for VCFA

Thus we need a Supervisor configured on the vSphere Cluster where DSM resources should reside. For that to work we need the DSM Supervisor Service (Database Consumption Operator).

## Register DSM Supervisor Service

A Supervisor Service leverages vSphere Pods (CRX Runtime). This service allows the VCFA-DSM extension to consume the Supervisor.



> [Supervisor Services TechDocs](https://techdocs.broadcom.com/us/en/vmware-cis/vcf/vcf-9-0-and-later/9-0/overview-of-vmware-cloud-foundation-9/what-is-vmware-cloud-foundation-and-vmware-vsphere-foundation/vcf-automation-overview/vsphere-supervisor.html)



We need two YAML files for that:
Download the package.yaml and values.yaml files.
Navigate to https://support.broadcom.com/group/ecx/productdownloads?subfamily=vSphere%20Supervisor%20Services.

- package.yaml - is the service
- values.yaml - is the configuration

values.yaml EXAMPLE

```
# imagePullSecret can be removed as anonymous image pull is currently supported by the default image repository
imagePullSecret: registry-creds
# imagePullSecretGeneration can be removed as anonymous image pull is currently supported by the default image repository
imagePullSecretGeneration:
  create:
    password: password
    server: artifactory repo url
    username: username
dsm:
  authSecretName: dsm-auth-creds
  authSecretGeneration:
    create:
      endpoint: https://vcf-mgmt-dsm01.vcf.soultec.lab
      user: dsmadmin@vcf.soultec.lab
      password: VMware1!VMware1!
      rootCA: |
        -----BEGIN CERTIFICATE-----
        MIIEWDCCA0CgAwIBAgIGAZzrEACNMA0GCSqGSIb3DQEBCwUAMFkxCzAJBgNVBAYT
        AlVTMQwwCgYDVQQKDANEU00xFTATBgNVBAsMDERTTSBQcm92aWRlcjElMCMGCSqG
        SIb3DQEJARYWZHNtX3N1cHBvcnRAdm13YXJlLmNvbTAeFw0yNjAzMTEwNjM3MDBa
        Fw0zNjAzMTQwNjM3MDBaMIGhMQswCQYDVQQGEwJVUzEMMAoGA1UECgwDRFNNMSAw
        HgYDVQQLDBdEU00gUHJvdmlkZXIgTWdtdCBQbGFuZTESMBAGA1UEBwwJUGFsbyBB
        bHRvMScwJQYDVQQDDB52Y2YtbWdtdC1kc20wMS52Y2Yuc291bHRlYy5sYWIxJTAj
        BgkqhkiG9w0BCQEWFmRzbV9zdXBwb3J0QHZtd2FyZS5jb20wggEiMA0GCSqGSIb3
        DQEBAQUAA4IBDwAwggEKAoIBAQC0OOLaJD6ktfs47NwaaxeU3SF1zOUgweF/u1h4
        nfzZ1YVkaVgpAyPFuLc+42pPK0IQY4r8Gb96T5NdqlF5HglRJIdiADZTSlne8cDF
        jt6sSMMRg4Ke+Pd6i1OOpR9uOVdvFxW1aZjCNbYoWxcbg11R6qFh4k0zEWxfZW2u
        2ZsVSe/DBytkR+dsEUijnMRtfqTMx554RjvyB0UJbHNjnxnypC1SHBLLERVAH5d9
        +j8O/PoHLeRKHiQ/wLuaR/pV0cqdkAAUTj09p2IKmzlcXYMFUkYpwZYuGwczPgJV
        aQytGVvEiq547f3WDpXGWs7U8Uhw6PS9nQ363Ytcxfh+6Tc7AgMBAAGjgdwwgdkw
        HQYDVR0OBBYEFMulbQPQnZehTUSS3ukzR82v2i1gMIGGBgNVHSMEfzB9gBQ6hDf1
        eM/HhSpjkqyejSqesEvbI6FdpFswWTELMAkGA1UEBhMCVVMxDDAKBgNVBAoMA0RT
        TTEVMBMGA1UECwwMRFNNIFByb3ZpZGVyMSUwIwYJKoZIhvcNAQkBFhZkc21fc3Vw
        cG9ydEB2bXdhcmUuY29tggYBnOsP8wYwLwYDVR0RBCgwJocEChgym4IedmNmLW1n
        bXQtZHNtMDEudmNmLnNvdWx0ZWMubGFiMA0GCSqGSIb3DQEBCwUAA4IBAQA2IerC
        NQTHi5ePoTHu3zGt4n10khkNTHfvKUZQQ54wYWZ12gM3DZ3WE5wHkAv8c3KuuG23
        s5OLcs9VgCqXUtaAywNphHYkEHLcORy9QY0NS6YxwaL60QmYNvMnTAFYrSZbp0Pm
        H/o2hnbq5QDzlcb57pyN7y11Y+bV0sASNmid2PDb5r55u0YZbixu8TBTXepGqZ9S
        Wyh8V3bu+ziN4vCwyUCnVIImPDByFvkpLaqFVMbvTxGL4dNfkWq4oEaspiW90CXK
        O9J71qm2fiprEENa9yuUBB1aw5zf5Fc+vGLldo084d52nvTefsnB88geML0v8Mnc
        XO84dp4fz69x2EDD
        -----END CERTIFICATE-----

  isSupervisor: true
consumptionClusterName: vcf-mgmt-cl01-sn01

```

Registering the service happens via the vSphere UI:

<img src="/images/vcfa-dsm/dsm-supervisor-service-1.png" width="1100" alt="Register DSM Supervisor Service step 1" title="VMware Data Services Manager" />

<img src="/images/vcfa-dsm/dsm-supervisor-service-2.png" width="1100" alt="Register DSM Supervisor Service step 2" title="VMware Data Services Manager" />

<img src="/images/vcfa-dsm/dsm-supervisor-service-3.png" width="1100" alt="Register DSM Supervisor Service step 3" title="VMware Data Services Manager" />





Further information: https://techdocs.broadcom.com/us/en/vmware-cis/dsm/data-services-manager/9-0/using-vmware-data-services-manager-with-vmware-cloud-foundation/using-as-a-provider/install-the-dsm-supervisor-service.html


# Register VCFA DSM Service

Now we need to connect VCFA to DSM. For that, log in to VCFA as a Provider Admin:

+NEW CONNECTION

<img src="/images/vcfa-dsm/dsm-vcfa-connection-1.png" width="1100" alt="VCFA DSM new connection" title="VMware Data Services Manager" />

And fill out the details (DSM cert can be downloaded via browser):

<img src="/images/vcfa-dsm/dsm-vcfa-connection-2.png" width="1100" alt="VCFA DSM connection details" title="VMware Data Services Manager" />



## Create VCFA Policy

Now we create a policy that defines in which vSphere Namespace and VCFA Organization DSM can be consumed:

<img src="/images/vcfa-dsm/dsm-vcfa-policy-1.png" width="1100" alt="VCFA DSM policy creation" title="VMware Data Services Manager" />

<img src="/images/vcfa-dsm/dsm-vcfa-policy-2.png" width="1100" alt="VCFA DSM policy details" title="VMware Data Services Manager" />

Also check in vSphere UI if the vSphere Namespace is ready:


<img src="/images/vcfa-dsm/dsm-vsphere-ns-1.png" width="1100" alt="vSphere Namespace ready" title="VMware Data Services Manager" />


# Create a DBaaS

Log in to VCFA as an Org or Project Admin:

Provision a Postgres cluster:

<img src="/images/vcfa-dsm/vcfa-1.png" width="1100" alt="VCFA Postgres provisioning" title="VMware Data Services Manager" />



Fill out the details:

<img src="/images/vcfa-dsm/vcfa-2.png" width="1100" alt="DSM" title="VMware Data Services Manager" />

# Log in to Postgres Cluster

Now the Postgres service is provisioned through VCFA. To connect to the database, use for example pgAdmin:

<img src="/images/vcfa-dsm/dsm-ready.png" width="1100" alt="DSM database ready" title="VMware Data Services Manager" />

<img src="/images/vcfa-dsm/dsm-ready-pgadmin.png" width="1100" alt="pgAdmin connection to DSM database" title="VMware Data Services Manager" />

<img src="/images/vcfa-dsm/dsm-ready-pgadmin-2.png" width="1100" alt="pgAdmin connected to DSM database" title="VMware Data Services Manager" />

Within the DSM UI, the database cannot be edited as it was deployed through VCFA and the Kubernetes Consumption Operator.

<img src="/images/vcfa-dsm/dsm-99.png" width="1100" alt="DSM UI read-only view" title="VMware Data Services Manager" />

<img src="/images/vcfa-dsm/dsm-100.png" width="1100" alt="DSM UI managed by VCFA" title="VMware Data Services Manager" />




# Conclusion

Setting up DSM with VCFA is quite easy and straightforward. As with everything provisioned via VCFA, you can use the YAML file for the DBaaS in your CI/CD pipeline. If you want to provision DBs from within your K8s guest cluster (and still have the DBs running outside of them), see my post about the DSM Kubernetes Operator: https://ygerber.online/post/dsm-9-k8s/

TechDocs article for DSM with VCFA:

https://techdocs.broadcom.com/us/en/vmware-cis/dsm/data-services-manager/9-0/using-vmware-data-services-manager-with-vmware-cloud-foundation/using-as-a-provider.html