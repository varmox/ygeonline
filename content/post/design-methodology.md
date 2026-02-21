---
title: "Design Methodology" # Title of the blog post.
date: 2026-01-21T00:39:48+01:00 # Date of post creation.
description: "The three tier design methodology" # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: true # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
usePageBundles: false # Set to true to group assets like images in the same folder as this post.
featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
featureImageAlt: 'Description of image' # Alternative text for featured image.
featureImageCap: 'This is the featured image.' # Caption (optional).
thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Technology
tags:
  - Architecture
  - Design
# comment: false # Disable comment if false.
---

**Insert Lead paragraph here.**

In Technical Architecture the three-tier oder conceptual model comes often into play. You probably stumbled into a few key words when reading VMware Docs or Reference Guides. Words like:

- logcal design
- physical design
- conceptual design

# The three Layers

## Physical Design

> the HOW

The physical design is probably the easiest layer where us engineers can relate most. It defines HOW the system is built. A few examples for the physical design:

- specific server models and their hardware layout, BIOS/UEFI Config
- Which Switchports are used
- vSAN Disk Layout
- Physical Rack Layout
- etc.
- configuration details (like HA, DRS, ESX Advanced Settings)

If you compare it with a house, the physical architecture is the CAD Drawing:

Output: LLD (Low Level Design), rack diagrams, IP sheets, runbooks.

## Conceptual Design

> the WHY

The conceptual design is the opposite of the physical design, is all about the why - the "owners perspective". No vendor names, no specs. Pure intent and requirements.

"We need high availability for critical workloads"
"Development environments must be isolated from production"
"We need DR capabilities with RTO < 4h"
"Infrastructure must scale to support 500 VMs over 3 years"

## Logical Design

The logical design glues the physical and conceptual design together. The logical design turns vague ideas into a clear architectural blueprint – this is the architect's perspective.

It's where design decisions are made explicit: what are the benefits, what are the risks, why was one approach chosen over another.

Without a logical design, things break down in both directions. A conceptual design alone leaves too many possible technical solutions open – there's no basis for choosing between them. A physical design alone is just a set of instructions with no justification behind them – no one can trace back why things were built that way or connect them to any business requirement.
That's why the logical design is arguably the most important part of any architecture. It bridges the gap between business and engineering. The owner can read it and see their requirements reflected. The engineer can read it and understand exactly what to build and why. It's the one layer that both sides can understand and relate to.

Its Vendor-aware but not yet hardware-specific.

- vSphere cluster topology: how many clusters, what separation (management / edge / compute)
- vSAN vs. shared storage decision
- NSX segment and tier layout: T0/T1 design, overlay vs. underlay separation
- Network zones: management, vMotion, vSAN, VM traffic – VLANs defined but no physical ports yet
- vCenter hierarchy: datacenter objects, host folders, resource pools, tagging strategy
- DRS / HA policies per cluster
- DR strategy: Stretched cluster vs. SRM vs. Zerto

Output: Logical diagrams, VLAN tables, cluster design docs – the classic VMware HLD (High Level Design)


# Important Categories

Now that we have the three layers whats very important and what should always be documented in a project are the following categories:

- Requirements (Functional and non-functional)
- Contraints
- Assumptions
- Risks

When working on a project design decisions should 