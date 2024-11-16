---
title: "[Guide] vSphere REST API - Intro" # Title of the blog post.
date: 2024-06-20T22:22:08+02:00 # Date of post creation.
description: "Introduction to the vSphere REST API" # Description used for search engine.
featured: false # Sets if post is a featured post, making appear on the home page side bar.
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
  - vSphere
  - Automation
# comment: false # Disable comment if false.
---

# Getting started with the vSphere API
The vSphere REST API, introduced in vSphere 6.5, represents a significant leap forward in managing and automating VMware environments. This modern, developer-friendly interface offers a more streamlined approach to interacting with vSphere compared to its predecessor, the SOAP-based vSphere Web Services API.

Key features of the vSphere REST API include:

- RESTful architecture: It follows REST principles, using standard HTTP methods (GET, POST, PUT, DELETE) for operations.
- JSON-based: Requests and responses use JSON format, making it easier to parse and work with data.
- Simplified interaction: The API is designed to be more intuitive and easier to use than the older SOAP-based API.


To get started with the vSphere REST API, you can use the built-in API Explorer in vCenter Server, accessible at https://< vcenter.example.com >/apiexplorer. This tool provides interactive documentation and allows you to test API calls directly from your browser.

## Endpoints

The vSphere API provides several key endpoints for interacting with different aspects of the vSphere environment:

| API  |  Endpoint |  
|---|---|
|  vSphere Automation API (REST) | https://< vcenter.example.com >/api  |  
|  VIM JSON API (New in vSphere 8.0 Update 1) | http://< vcenter.example.com >/sdk/vim25/8.0.1.0  |   
|  vSphere Web Services API (SOAP) - deprecated | https://< vcenter.example.com >/sdk  |  
|  vCenter Server Appliance Management API | https://< vcenter.example.com >:5480/rest  |  
|  API Explorer: | https://< vcenter.example.com >/apiexplorer  |  


![vSphere API Endpoints Overview](https://i.imgur.com/iP0EE4h.png)

## vSphere Developer Center

Within your vCenter there a some tools to help you interacting with the vSphere API provided by your vCenter. So you only use API Call that are actually available for your infrastructure.

### Code Capture

With Code Capature you can capture a manual configuration and vSphere will give you the code in various languages for that configuration.

***Activate Code Capture***

From the burger menu, click Developer Center and go to the Code Capture tab. Now a Red Recording Buttons appears on the top right. To start a recording, navigate to your desired pane and click the red record button in the top pane. To start recording immediately, click Start Recording.
While a recording is in progress, the red record button in the top pane blinks.

While the recording is in progress, do your things in the GUI (like edit a VM, add a second Disk). After your manual configuration has been made, stop the recording. Now you will be presented with a sample Code.

### API Explorer

To explore the vSphere APIs, you can access the API Explorer directly in your vCenter.
From the burger menu, click Developer Center and select the API Explorer tab.

Or Directly via: https://< vcenter.example.com >/apiexplorer

## Ansible

Ansible can interact with vSphere via Module (mostly the Community.Vmware' module is used). But not all Automation Tasks can be done via this module. Sometimes direct API Calls via the 'uri' module have to be made for certain tasks.

### Example Ansible vSphere REST API Call

```
- name: Basic vSphere API REST Call
  hosts: localhost
  gather_facts: no
  vars:
    vcenter_hostname: "vcenter.example.com"
    vcenter_username: "administrator@vsphere.local"
    vcenter_password: "your_password_here"

  tasks:
    - name: Get vCenter session token
      uri:
        url: "https://{{ vcenter_hostname }}/rest/com/vmware/cis/session"
        method: POST
        validate_certs: no
        force_basic_auth: yes
        user: "{{ vcenter_username }}"
        password: "{{ vcenter_password }}"
      register: login_result
```

#### vSphere managedObject ID
The Managed Object ID (MOID), also known as the Managed Object Reference ID (MORef ID), is a VMware internal identifier that is generated by vSphere when new objects like VMs are created, or when ESXi hosts are added to vCenter. MOIDs are used to uniquely identify VMware components and are used by all VMware solutions to reference objects within vCenter.

MOIDs typically consist of a prefix indicating the object type, followed by a number. For example, "vm-1024" for a virtual machine or "datacenter-1001" for a datacenter

A lot of times you will need to interact with the MOIDs. You can browse them in your vCenter:

- https://< vcenter.example.com >/mob


To get those MOIDs here is an example:

```
---
- name: Gather ESXi Host Info using vSphere API
  hosts: localhost
  gather_facts: no
  vars:
    vcenter_hostname: "vcenter.example.com"
    vcenter_username: "administrator@vsphere.local"
    vcenter_password: "your_password_here"
    cluster_name: "your_cluster_name"

  tasks:
    - name: Get vCenter session token
      uri:
        url: "https://{{ vcenter_hostname }}/rest/com/vmware/cis/session"
        method: POST
        validate_certs: no
        force_basic_auth: yes
        user: "{{ vcenter_username }}"
        password: "{{ vcenter_password }}"
      register: login_result

    - name: Get cluster MoID
      uri:
        url: "https://{{ vcenter_hostname }}/rest/vcenter/cluster"
        method: GET
        validate_certs: no
        headers:
          vmware-api-session-id: "{{ login_result.json.value }}"
      register: clusters_result

    - name: Set cluster MoID
      set_fact:
        cluster_moid: "{{ clusters_result.json.value | selectattr('name', 'equalto', cluster_name) | map(attribute='cluster') | first }}"

    - name: Get ESXi hosts in cluster
      uri:
        url: "https://{{ vcenter_hostname }}/rest/vcenter/host?filter.clusters={{ cluster_moid }}"
        method: GET
        validate_certs: no
        headers:
          vmware-api-session-id: "{{ login_result.json.value }}"
      register: hosts_result

    - name: Display ESXi host information
      debug:
        msg: "Host Name: {{ item.name }}, Host MoID: {{ item.host }}"
      loop: "{{ hosts_result.json.value }}"

    - name: Get detailed host info
      uri:
        url: "https://{{ vcenter_hostname }}/rest/vcenter/host/{{ item.host }}"
        method: GET
        validate_certs: no
        headers:
          vmware-api-session-id: "{{ login_result.json.value }}"
      register: host_details
      loop: "{{ hosts_result.json.value }}"

    - name: Display detailed host information
      debug:
        msg: "Host: {{ item.json.value.name }}, Connection State: {{ item.json.value.connection_state }}, Power State: {{ item.json.value.power_state }}"
      loop: "{{ host_details.results }}"
```


This approach using the uri module gives you more direct control over the API calls and allows you to customize the requests and responses as needed. It's particularly useful when you need to perform actions that aren't covered by existing Ansible modules or when you want to work directly with the vSphere REST API.

## Powershell

If PowerCLI does not have the function you are looking for, you could also you native Powershell to interact with the vSphere API.

This script does the following:
- Sets up the vCenter server details.
- Adds a function to ignore SSL certificate errors (remove this in production environments).
- Authenticates with the vCenter server to get a session token.
- Uses the session token to make an API call to retrieve a list of VMs.
- Displays the results in a table format.


```
# vCenter server details
$vcServer = "vcenter.example.com"
$vcUser = "administrator@vsphere.local"
$vcPass = "YourPassword"

# Ignore SSL certificate errors (remove in production)
if (-not ([System.Management.Automation.PSTypeName]'ServerCertificateValidationCallback').Type) {
    add-type @"
    using System.Net;
    using System.Security.Cryptography.X509Certificates;
    public class ServerCertificateValidationCallback {
        public static void Ignore() {
            ServicePointManager.ServerCertificateValidationCallback += 
                delegate (
                    Object obj, 
                    X509Certificate certificate, 
                    X509Chain chain, 
                    SslPolicyErrors errors
                ) { return true; };
        }
    }
"@
}
[ServerCertificateValidationCallback]::Ignore()

# Authenticate and get session token
$auth = [System.Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes($vcUser + ':' + $vcPass))
$headers = @{
    'Authorization' = "Basic $auth"
}
$sessionUrl = "https://$vcServer/rest/com/vmware/cis/session"
$response = Invoke-RestMethod -Uri $sessionUrl -Method Post -Headers $headers
$sessionId = $response.value

# Set up headers for subsequent requests
$headers = @{
    'vmware-api-session-id' = $sessionId
}

# Make an API call (e.g., get list of VMs)
$vmsUrl = "https://$vcServer/rest/vcenter/vm"
$vms = Invoke-RestMethod -Uri $vmsUrl -Method Get -Headers $headers

# Display results
$vms.value | Format-Table
```