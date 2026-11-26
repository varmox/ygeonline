---
title: "Reformatting enterprise storage drives to 512 bytes" # Title of the blog post.
postType: "HowTo"
date: 2026-11-26T01:30:00+02:00 # Date of post creation.
description: "use enterprise storage drives elsewhere." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: ßßß
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
  - Technology
tags:
  - Hardware
  - HPE 
# comment: false # Disable comment if false.
---

**Problem**

Recently I got my hands on some old HPE 3PAR 1.92TB SAS SSDs (Samsung PM1633a). I though they would be perfect for my SDDC home lab - sadly they did not work as intended with my HPE DL360 server. The SmartArray P420i was recognizing the disks but was giving the error "This physical drive does not support RAID and is not exposed to OS. It cannot be used for configuration on this controller". But hopefully there is a solution to the problem we are facing:

Basically the drives in a 3PAR Storage System use 520Bytes blocks as a low-level formatting. We need 512 Bytes for standard Operating Systems like Linux (520 Bytes does enable T10 DIF CRC error checking - the extra 8 Bytes are designated for data integrity/protection [1])


**Solution**

***TL;DR:***

- connect to SAS Drives to a HBA or RAID-Controller in HBA Mode
- Install a OS (Windows or Linux) on a seperate, already working drive - or just use a live version
- install sg3_utils package [2]
- run 𝙨𝙜_𝙨𝙘𝙖𝙣 and 𝒔𝒈_𝒇𝒐𝒓𝒎𝒂𝒕 --𝒇𝒐𝒓𝒎𝒂𝒕 --𝒔𝒊𝒛𝒆 512 𝑷𝑫𝒙  to reformat the disks to 512 Byte blocks



The first step is to install the disk in a system with only a simple SAS HBA - most raid adapters will give you errors when you want to expose the disk to the OS. We need SCSI Access from a OS to do some commands. I used a HPE P420i in HBA Mode (Note: to be able to set your RAID Controller to HBA Mode, no logical drives should be present)

Next install a OS on a already working drive, I used Rocky Linux - a live version also works fine. To be able to format the drives we need sg3_utils. Simply install the package with dnf:

```
sudo dnf install sg3_utils
```


Then with the help of sg3_utils package we run some commands. First we need to list all SCSI drives and get their ID (PDx)

***Windows***

```
sg_scan
```
Then format one drive at a time (in this case PD1)

```
sg_format --format --size 512 PD1
```

***Linux***

```
sg_scan -i
```
Then format one drive at a time (in this case sg3)

```
sg_format --format --size 512 /dev/sg3
```

Now the disks are formatted with 512 byte blocks and you should be able to use the disk in a JBOD or RAID.

**Update (2026): when your RAID controller refuses HBA mode**

The method above needs the controller in HBA/JBOD mode. But some RAID controllers are firmware-locked to RAID mode and simply won't let you switch. I ran into this with a Lenovo ThinkSystem RAID 930-8i (Broadcom SAS3508) and three Samsung 1.92TB SAS SSDs pulled from an old array - same 520-byte problem as before, but this time there was no way into HBA mode:

- `storcli /c0 set personality=HBA` → *"Controller does not support setting personality"*
- in the UEFI setup the **Personality Mode** dropdown is greyed out, even after deleting every virtual drive
- cross-flashing or the legacy Ctrl+R BIOS is a dead end too - it's the same firmware underneath

So the drives show up as `UGUnsp` (Unconfigured Good - Unsupported), 0 KB, and never appear as `/dev/sg*` - `sg_format` has nothing to point at, and `set jbod`/`set good` both return *"Operation not allowed"*.

***Solution:*** send the `FORMAT UNIT` command *straight through* the RAID controller's management interface, with [megaraid_format_tools](https://github.com/aronfirebot/megaraid_format_tools) [3]. It talks to `/dev/megaraid_sas_ioctl_node` (the same passthrough `smartctl -d megaraid,N` uses), so it can reach a drive the OS can't otherwise see. No HBA mode, no personality change - the controller stays in RAID mode the whole time.

***TL;DR:***

- make any spare drive a JBOD so you have one `/dev/sd*` on the controller (the tool uses it only to find the host number)
- get each target's id - the **DID** column, *not* the slot - from `storcli /c0/eall/sall show`
- confirm the passthrough reaches the drive (read-only): `smartctl -d megaraid,<id> -i /dev/sda`
- build the tool and run `sudo ./mega_format_immed /dev/sda <id> 512` for each drive
- wait (~20 min per SSD), verify with smartctl, then **reboot** so the controller re-reads the drives
- `set good` the drives and build your RAID

First, give yourself a handle. The tool needs any one `/dev/sd*` on the same controller just to look up the host number (it's never written to). If there are no virtual drives, turn a spare drive into a JBOD:

```
storcli /c0/e134/s0 set jbod        # now shows up as /dev/sda
ls -l /dev/megaraid_sas_ioctl_node  # must exist (megaraid_sas driver)
```

Find the target id of each drive to format. Use the **DID** column, not the slot number:

```
storcli /c0/eall/sall show
```

Before touching anything, prove the controller will pass commands through to the drive. This is read-only - if it prints the drive's identity, `FORMAT UNIT` will work too:

```
smartctl -d megaraid,2 -i /dev/sda   # "Logical block size: 520 bytes"
```

Now build the tool (`build-essential` + `smartmontools`) and reformat each drive. It sends `INQUIRY` → `MODE SELECT` (set 512) → `FORMAT UNIT` with the IMMED bit, so it returns immediately and the drive formats in the background:

```
git clone https://github.com/aronfirebot/megaraid_format_tools
cd megaraid_format_tools && make
sudo ./mega_format_immed /dev/sda 2 512   # repeat per target id
```

The format runs in the background. `smartctl` reads the drive live, so it's your truth for "is it 512 yet" - it flipped after ~20 minutes per SSD for me:

```
smartctl -d megaraid,2 -i /dev/sda | grep "block size"   # -> 512 bytes
```

Here's the gotcha: even once the drives are 512, the controller keeps serving its old cached `UGUnsp / 0 KB` view, and `set good` still fails. A **reboot** fixes it - the controller re-scans and finally sees the drives as valid 512-byte disks. After that:

```
for s in 2 3 4; do storcli /c0/e134/s$s set good force; done
storcli /c0 add vd type=raid0 drives=134:2,134:3,134:4
```

And the drives that *"cannot be used for configuration on this controller"* are now a healthy RAID volume - without ever leaving RAID mode.

***References***

[1] https://www.openfabrics.org/images/2018workshop/presentations/307_TOved_T10-DIFOffload.pdf 

[2] https://sg.danny.cz/sg/sg3_utils.html 

[3] https://github.com/aronfirebot/megaraid_format_tools 