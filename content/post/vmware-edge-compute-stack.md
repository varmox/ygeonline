---
title: "VMware Edge Compute Stack" # Title of the blog post.
date: 2024-12-31T18:52:03+01:00 # Date of post creation.
description: "Intro to VMware Edge Compute Stack (ECS)." # Description used for search engine.
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
  - VMware
  - Kubernetes
tags:
  - VMware ECS
  - VMware VECO
# comment: false # Disable comment if false.
---

# Intro

VMware Edge Cloud Compute Stack is a Solution for running workloads on the Edge - "Edge" in this case means running a modified Version of ESXi on really small compute footprint.

## Management

Unlinke vSphere you won't deploy a vCenter to manage your ECS Hosts. In a classic vSphere Infrastructure the vCenter is Control- and Management Plane. In ECS your Management Plane is the VMware Edge Cloud Orchestrator (VECO). This Appliance runs either as a SaaS Service (on Google Cloud) or can be deployed on-premises.

Control Plane Components are always residing on the Edge Site, so you do not need constant connectivity to VECO.

## Hardware

ECS Hosts (a modified Version of ESXi, but with full ESXi Capabilities) can be even [consumer Hardware](https://compatibilityguide.broadcom.com/detail?program=cpu&productId=150&persona=live&column=cpuSeries&order=desc&activeDelta=100&redirectFrom=AMD%20Ryzen%20Embedded%20V1000%20Series) with very small energy consumption. Like Barebones, NUCs, etc. You do not need beefy enterprise servers.

You can even have [USB to RJ45 Network Adapters](https://github.com/alanrenouf/ECSExample/blob/main/edge-host-config/usb-nic-scan-on-boot.yaml) with the [VMware USB Network Driver Fling](https://community.broadcom.com/flings/home)!

Just follow the VMware VGC for [supported Hardware](https://compatibilityguide.broadcom.com/).

## GitOps Approach

The unique thing about ECS is you manage the ECS Hosts and the Workloads with a GitOps approach. The configuration of the ECS Hosts like Networking, vSwitch Config is defined via yaml and stored in a Git Repository. The ECS Hosts has FluxCD installed on will check within the GitRepo for new yaml Manifests. FluxCD has a pull based approach - the destination (the ECS Host) will pull the config (yaml Files) themselves. With this approach it isn't a big Problem if the Edge Site does not have constant connectivity.

[VMware ECS Big Picture - Copyright by VMware by Broadcom](https://blogs.vmware.com/sase/files/2024/06/blog-ECS-3-5-release-graphic-1.png)

You cannot modify anything manually, the ECS Hosts will only have the GitRepo as a Configuration Source. 

**Example**

```
#Example host configuration adding a new vSwitch and portgroup and assigning them to vmnic1
apiVersion: esx.vmware.com/v1alpha1
kind: HostConfiguration
metadata:
  name: vswitch-config
  namespace: esx-system
spec:
  layertype: Incremental
  profile: |
    {
        "esx": {
            "network_vss": {
                "switches": [
                    {
                        "name": "vSwitch1",
                        "policy": {
                            "nic_teaming": {
                                "policy": "LOADBALANCE_SRCID",
                                "notify_switches": true,
                                "rolling_order": false,
                                "link_criteria_beacon": "IGNORE",
                                "active_nics": [
                                    "vmnic1"
                                ],
                                "standby_nics": []
                            },
                            "security": {
                                "allow_promiscuous": false,
                                "mac_changes": false,
                                "forged_transmits": false
                            },
                            "traffic_shaping": {
                                "enabled": false
                            }
                        },
                        "bridge": {
                            "link_discovery_protocol": {
                                "protocol": "CDP",
                                "operation": "LISTEN"
                            },
                            "nics": [
                                "vmnic1"
                            ]
                        },
                        "num_ports": 1024,
                        "mtu": 1500,
                        "port_groups": [
                            {
                                "name": "VM Network 16",
                                "vlan_id": 0
                            }
                        ]
                    }
                ]
            }
        }
    }
```

## Workloads

Workloads on ECS can either be classical VMs or Kubernetes Workloads.

## Networking

LAN Connectivity will rely on vSwitches, NSX isn't designed for ECS as the footprint is too big.
But for the WAN Connectivty, well there is a really good solution: VeloCloud SD-WAN

## SD-WAN

VeloCloud SD-WAN can either be deployed as a Virtual Appliance or Hardware.
VeloCloud allows for SD-WAN Connectivity, L3+L4+L7 Firewall and VPN Services as well as a lot of other features. VeloCloud SD-WAN deploys so called "Edges", the Appliance that will handle the traffic (Data Plane). The Management will be done from the SD-WAN Orchestrator Appliance, either running onpremises or as a SaaS (running on Google Cloud).



## Clusters

As of now two topologies are supported in a Edge Site:

- One Node 
- Two Node

I heard some rumors that three Nodes ESC Setup (event with vSAN) is comming ;)
