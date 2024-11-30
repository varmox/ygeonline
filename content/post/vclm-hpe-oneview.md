---
title: "[Guide] vSphere Lifecycle Manager and HPE OneView for vCenter - Troubleshooting" # Title of the blog post.
date: 2024-11-20T09:25:35+01:00 # Date of post creation.
description: "vLCM & HPE OneView" # Description used for search engine.
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
  - vSphere
  - HPE OneView
# comment: false # Disable comment if false.
---
# vSphere Lifecycle Manager (vLCM)

Some Guidance when troubleshooting Issues with vSphere Lifecycle Manager and HPE OneView for vCenter as a HSM (Hardware Support Manager).

## Log Files

The General Log File for vLCM is here:

Log File: /var/log/vmware/vmware-updatemgr/vum-server/vmware-vum-server.log

To see error from this log file

```
cat /var/log/vmware/vmware-updatemgr/vum-server/vmware-vum-server.log | grep "error"
```


3rd Party Depots and their the vCenter Access to those depot is logged in the following file:

```
cat /var/log/vmware/envoy/envoy-access.log | grep "hpe"
```


## Common Errors

### Cannot sync software depots

To verify that the online depot registration was successful, navigate to *Menu > Lifecycle Manager > Settings > Administration > Patch Setup*. The values in the Enabled and Connectivity Status columns should be *Yes* and *Connected* respectively. If the Connectivity Status is Not Connected, verify the proper settings for the vCenter proxy configuration and perform a manual sync of the updates.


Also get the log file:

```
cat /var/log/vmware/vmware-updatemgr/vum-server/vmware-vum-server.log 
```

If you see this error "A depot is inaccessible or has invalid contents. Make sure an official depot source is used and verify connection to the depot"

```
-->     "error_type": "ERROR",
-->     "messages": [
-->         {
-->             "args": [],
-->             "default_message": "A depot is inaccessible or has invalid contents. Make sure an official depot source is used and verify connection to the depot.",
-->             "id": "com.vmware.vcIntegrity.lifecycle.depotContent.ValidationError"
-->         }
-->     ]
--> }"
--> }
```

Check if you can access VMware Online Depots, from vCenter run:

```
 curl -vvv https://hostupdate.vmware.com
```

You should see a DigiCert Certificate printed:

```
*  issuer: C=US; O=DigiCert Inc; CN=DigiCert TLS RSA SHA256 2020 CA1
```

If not, check with your Firewall Team if they do TLS Intercepts.

Check also the connection to the OneView Depot

```
 curl -vvv https://oneviewforvcenter.domain.example
 curl -vvv https://oneviewforvcenter.domain.example:3512
 ```

Also check your connected depots. Maybe there is a old depot still configured.



### Running behind a HTTP(S) Forward Proxy

If your Infrastructure needs a forward proxy to access the internet, the following must be done at the vCenter Level.

- vCenter will connect to the Internet (hostupdate.vmware.com) for ESXi updates
- vCenter wil connect to the HPE OneView for vCenter Depots for all Firmware stuff (SPP)

So we need to set the HPE OneView in the NO_Proxy settings of the vCenter:

```
 vi /etc/sysconfig/proxy
 ```

 ```
# Example: NO_PROXY="internal.domain, internal-subnet , localhost"
NO_PROXY="localhost, 127.0.0.1, oneviewforvcenter.domain.example, IP of the HPE-OneView"
 ```

### HPE OneView for vCenter with named Certificates 

Check

```
cat /var/log/vmware/envoy/envoy-access.log | grep "hpe"
```

If you see some SSL Errors like:

```
failed: cURL Error: SSL peer certificate or SSH remote key was not OK, SSL certificate problem: certificate has expired
```

Verify if the required certs for HPE OneView for vCenter are valid.

[Further Troubleshooting Information](https://support.hpe.com/hpesc/public/docDisplay?docId=sd00002167en_us&docLocale=en_US&page=GUID-ACF7270C-14CD-41A6-B02A-2FA0EE0C4723.html)
