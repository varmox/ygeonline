---
title: "Reformatting enterprise storage drives to 512 bytes" # Title of the blog post.
date: 2023-12-26T15:28:45+02:00 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: ßßß
usePageBundles: false # Set to true to group assets like images in the same folder as this post.
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
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
  - Hardware
  - HPE 
# comment: false # Disable comment if false.
---

**Problem**

Recently I got my hands on some old HPE 3PAR 1.92TB SAS SSDs (Samsung PM1633a). I though they would be perfect for my SDDC home lab - sadly they did not work as intended with my HPE DL360 server. The SmartArray P420i was regocnizing the disks but was giving the error "This physical drive does not support RAID and is not exposed to OS. It cannot be used for configuration on this controller". But hopefully there is a solution to the problem we are facing:

Basically the drives in a 3PAR Storage System use 520Bytes blocks as a low-level formatting. We need 512 Bytes (520 Bytes does enable T10 DIF CRC error checking - the extra 8 Bytes are designated for data integrity/protection [1])


**Solution**

***TL;DR:***

- connect to SAS Drives to a HBA or RAID-Controller in HBA Mode
- Install a OS (Windows or Linux) on a seperate, already working drive
- install sg3_utils package [2]
- run 𝙨𝙜_𝙨𝙘𝙖𝙣 and 𝒔𝒈_𝒇𝒐𝒓𝒎𝒂𝒕 --𝒇𝒐𝒓𝒎𝒂𝒕 --𝒔𝒊𝒛𝒆 512 𝑷𝑫𝒙  to reformat the disks to 512 Byte blocks



The first step is to install the disk in a system with only a simple SAS HBA - most raid adapters will give you errors when you want to expose the disk to the OS. We need SCSI Access from a OS to do some commands. I used a HPE P420i in HBA Mode (Note: to be able to set your RAID Controller to HBA Mode, no logical drives should be present)

Next install a OS on a already working drive, I used Rocky Linux. To be able to format the drives we need sg3_utils. Simply install the package with dnf:

```
sudo dnf install sg3_utils.x86_64
```


Then with the help of sg3_utils package we run some commands. First we need to list all SCSI drives and get their ID (PDx)

```
sg_scan
```
Then format one drive at a time (in this case PD1)

```
sg_format --format --size 512 PD1
```

Now the disks are formatted with 512 byte blocks and you should be able to use the disk in a JBOD or RAID.

***References***

[1] https://www.openfabrics.org/images/2018workshop/presentations/307_TOved_T10-DIFOffload.pdf 

[2] https://sg.danny.cz/sg/sg3_utils.html 