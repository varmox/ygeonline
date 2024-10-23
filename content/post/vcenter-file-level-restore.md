---
title: "vCenter File Level Restore - Edit Configuration" # Title of the blog post.
date: 2024-02-23T12:18:49+02:00 # Date of post creation.
description: "Edit Configuratio before restoring a VCSA File-level Backup" # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
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
tags:
  - VMware 
  - vCenter
# comment: false # Disable comment if false.
---

# Intro

With vCenter File-level Backups you can restore your entire vCenter configuration. If your running vCenter has some misconfigurations (eg. a false proxy setting) and isn't working, you can use existing file-level backups and clear those misconfigurations before restoring.

## Basic Process

The Basic vCenter File-level Restore Process looks like the following:

- Deploy a fresh vCenter via ISO (VCSA UI Installer)
  - Shutdown the defunct vCenter (IP Conflicts)
  - Same Network Configuration
  - Same build number
  - Same Size (Tiny, Small, Medium Large)
- When Stage 1 is completed you can start the Stage 2 Process to Restore your vCenter Configuration 
- In case of a misconfiguration (which has also been backed up) you have the option to edit all configurations files
- Restore vCenter with configurations edits

# Stage 1 - Deploying a fresh (blank) appliance

Before deploying a fresh vCenter, have a look at you VCSA Backup folder. In the root of each backupfolder there is a file 'backup-metadata.json' - within there grab the details before deploying a fresh VCSA:

- "Build" (Use the exact same buildnumber)


After the successful deployment you with be presented with the Stage 2 Process. ('https://vcenterfqdn:5480/stage2')

# Stage 1.5 - Clear misconfigurations

Grab you desired the whole backupfolder from your desired date and safe it somewhere. As a better unterstanding of the folder Naming convention:

```
[M/S]_[VCSA version]_[Date YYYYMMDD]_[Time HHMMSS]_[Unique identifier]
```

M: Manual backup / S: Scheduled backup

file structure:

```
[SFTP/FTP/HTTP Server]
└── vCenter
    └── [VCSA FQDN]
        └── [M/S]_[VCSA version]_[Date YYYYMMDD]_[Time HHMMSS]_[Unique identifier]
            ├── backup.mf
            ├── backup_manifest.json
            ├── VADump.vad
            ├── config_files
            └── [Additional backup files]
```

It's important to note that the specific location of the backup files depends on the protocol and location you've configured for your VCSA backups, which can include FTPS, HTTPS, SFTP, FTP, NFS, SMB, or HTTP.

## Edit Configurations Files

The 'config_files' holds all the configuration files. You can safely edit those files "offline" from your workstation.

### Example - Edit HTTP Proxy

Unpack the archive:

```
tar -xvzf config_files.tar.gz
```

Edit the Proxy File:

```
vim /config_files/etc/sysconfig/proxy
```

Make your adjustments as needed. This is just an example. Within the config Files you could also edit Network Configs etc.

Then create the archive and gzip it:

```
tar -czvf archive_name.tar.gz config_files
```

Now upload the newly created archive to your SFTP/FTP/HTTP Server which the freshly deployed VCSA can access the folder.


### Encrypted Backups

If you enabled encryption in the file-level backups, those can be decrypted with openssl.

Example:
Config File Folder 'config_files.tar.gz.enc'

Decrypt the archive (provide the configured encryption key/passphrase as configured in the vcsa backup)

```
openssl aes-256-cbc -a -salt -pbkdf2 -in config_files.tar.gz.enc -out config_files.tar.gz
```

Unpack the archive as described above, make adjusments.

Then encrypt the files again:

```
openssl aes-256-cbc -a -salt -pbkdf2 -in config_files.tar.gz -out config_files.tar.gz.enc
```

# Stage 2 - File-level Restore

Now go back to 'https://vcenterfqdn:5480/stage2' and Start the Stage 2 Process. Provide the folder with the new edits.

Tipp: If you are unable to change the folder (Server, Location etc) at stage 2 - open 'https://vcenterfqdn:5480/stage2' in a private/incognito Windows in your Browser. Then you should be able to edit Stage 2 Restore Parameters.