---
title: "VCF FAQ"
date: 2026-04-15T22:07:06+02:00
description: "Common questions about VMware Cloud Foundation 9."
featured: true
draft: false
toc: false
usePageBundles: false
categories:
  - Technology
tags:
  - VCF
  - VMware
---

# VCF FAQ

<details>
<summary><strong>What is Greenfield and Brownfield?</strong></summary>

Greenfield is a fresh installation. Brownfield is taking your existing VMware infrastructure and migrating it to 9.
</details>

<details>
<summary><strong>Do I need SDDC Manager / Fleet Manager with VCF9?</strong></summary>

No — for VCF9 you just need VCF Operations.
</details>

<details>
<summary><strong>How do I upgrade from vSphere 8 to vSphere 9?</strong></summary>

TL;DR: Install VCF Ops (or upgrade Aria Ops 8.18 to 9). Then update vCenter and ESX as you have in previous years.

If you want to use all the Features that VCF Brings. Use VCF Installer. Is way easier.
</details>

<details>
<summary><strong>What's the VCF Installer doing then?</strong></summary>

VCF Installer does the upgrade automatically for you. No need for manual OVA provisioning etc. It also includes a lot of prebuilt checks (hardware compatibility, configuration issues, etc.).
</details>

<details>
<summary><strong>What is VCF Fleet Management?</strong></summary>

It's part of VCF Operations. The Fleet Manager will upgrade your whole VCF fleet (vCenter, NSX, Ops, ESX, SDDCm) for you. No manual patching needed. Lots of prebuilt checks.
</details>

<details>
<summary><strong>Do I need a Management Domain and Workload Domain?</strong></summary>

No. Your first domain will be your Management Domain. You can onboard additional Workload Domains as you wish.
</details>

<details>
<summary><strong>Is iSCSI still supported?</strong></summary>

Yes. For Brownfield it's supported. For Greenfield installations with the VCF Installer you have to do the iSCSI configuration beforehand.
</details>

<details>
<summary><strong>Do I need to run vSAN?</strong></summary>

No (but I encourage you to do so 😉).
</details>

<details>
<summary><strong>Does vSAN ESA now support deduplication?</strong></summary>

Yes.
</details>

<details>
<summary><strong>What's a Domain in the context of VCF?</strong></summary>

A Domain or Workload Domain is the boundary of a vCenter and SDDC Manager (SDDCm).
</details>

<details>
<summary><strong>Why the Management Domain then?</strong></summary>

Keeping Management and Workload separate keeps the blast radius small. Keep your VCF appliances, AD controllers, monitoring VMs etc. in your Management Domain — your application VMs in a separate Workload Domain.
</details>

<details>
<summary><strong>What's the minimum amount of servers for a VCF9 installation?</strong></summary>

With external block storage it's 2 servers. With vSAN it's 3.
</details>

<details>
<summary><strong>Is NSX mandatory?</strong></summary>

No. NSX will get deployed when you upgrade with the VCF Installer, but you don't need to use it.
</details>

<details>
<summary><strong>For what use cases is NSX mandatory then?</strong></summary>

If you want to use VCF Automation with VPCs (aka the "All-Apps" organization) you need NSX. That's the case with a self-service vSphere Kubernetes cluster.
</details>
