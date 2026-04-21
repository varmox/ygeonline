---
title: "Useful VMware Links"
date: 2026-04-10T20:24:29+02:00
description: "A curated collection of useful VMware links: official documentation, KBs, tools, and community resources."
featured: true
draft: true
toc: true
categories:
  - Technology
tags:
  - VMware
  - vSphere
  - VCF
  - VKS
  - Links
---

A curated collection of VMware resources I keep coming back to — official docs, KBs, tools, and community content. Grouped by product area for quick access.

<div id="vmware-links">

<p>
  <button type="button" id="export-bookmarks-btn"
          style="display:inline-block;padding:0.6em 1.2em;font-size:1rem;font-weight:600;color:#fff;background:#0091da;border:none;border-radius:4px;cursor:pointer;">
    ⬇ Download as Browser Bookmarks
  </button>
  <span style="margin-left:0.5em;font-size:0.9em;opacity:0.75;">
    (Netscape HTML format — import into Chrome, Firefox, Safari, Edge)
  </span>
</p>

<script>
(function () {
  var btn = document.getElementById('export-bookmarks-btn');
  if (!btn) return;

  btn.addEventListener('click', function () {
    var root = document.getElementById('vmware-links');
    if (!root) return;

    // Walk siblings after the wrapper in document order to collect H2 + following table pairs.
    // Because the wrapper only contains the button, the H2/tables live as siblings of #vmware-links
    // in the rendered post content. We scan the nearest content container instead.
    var container = root.parentElement;
    var nodes = container ? Array.from(container.children) : [];

    var folders = [];
    var current = null;
    nodes.forEach(function (el) {
      if (el.tagName === 'H2') {
        current = { name: el.textContent.trim(), links: [] };
        folders.push(current);
      } else if (el.tagName === 'TABLE' && current) {
        el.querySelectorAll('tbody tr').forEach(function (tr) {
          var a = tr.querySelector('a[href]');
          if (!a) return;
          var href = a.getAttribute('href');
          if (!href || href.indexOf('https://example.com') === 0) return; // skip placeholders
          current.links.push({ title: a.textContent.trim() || href, href: href });
        });
      }
    });

    var lines = [];
    lines.push('<!DOCTYPE NETSCAPE-Bookmark-file-1>');
    lines.push('<META HTTP-EQUIV="Content-Type" CONTENT="text/html; charset=UTF-8">');
    lines.push('<TITLE>Bookmarks</TITLE>');
    lines.push('<H1>Bookmarks</H1>');
    lines.push('<DL><p>');
    lines.push('    <DT><H3>VMware Links</H3>');
    lines.push('    <DL><p>');

    var esc = function (s) {
      return String(s).replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;').replace(/"/g, '&quot;');
    };

    folders.forEach(function (f) {
      if (!f.links.length) return;
      lines.push('        <DT><H3>' + esc(f.name) + '</H3>');
      lines.push('        <DL><p>');
      f.links.forEach(function (l) {
        lines.push('            <DT><A HREF="' + esc(l.href) + '">' + esc(l.title) + '</A>');
      });
      lines.push('        </DL><p>');
    });

    lines.push('    </DL><p>');
    lines.push('</DL><p>');

    var blob = new Blob([lines.join('\n')], { type: 'text/html;charset=utf-8' });
    var url = URL.createObjectURL(blob);
    var a = document.createElement('a');
    a.href = url;
    a.download = 'vmware-links.html';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    setTimeout(function () { URL.revokeObjectURL(url); }, 1000);
  });
})();
</script>

</div>

## VCF Design and Sizing

| Name | Description |
| --- | --- |
| [vSAN Sizer](https://example.com) | Short description. |
| [Private Cloud Sizer](https://example.com) | Short description. |

## VCF Converge Existing Infrastructre

| Name | Description |
| --- | --- |
| [Supported Scenarios](https://techdocs.broadcom.com/us/en/vmware-cis/vcf/vcf-9-0-and-later/9-0/deployment/converging-your-existing-vsphere-infrastructure-to-a-vcf-or-vvf-platform-/supported-scenarios-to-converge-to-vcf.html) | Upgrade from vSphere to 9 |


## VCF Preparation

| Name | Description |
| --- | --- |
| [Planning and Preparation](https://techdocs.broadcom.com/us/en/vmware-cis/vcf/vcf-9-0-and-later/9-0/planning-and-preparation.html) | Short description. |

## vSphere

| Name | Description |
| --- | --- |
| [Title](https://example.com) | Short description. |

## vSAN

| Name | Description |
| --- | --- |
| [vSAN Technical Guides](https://www.vmware.com/products/cloud-infrastructure/vsan/resources#technical-paper) | All vSAN Technical Guides |
| [vSAN Technology Overview](https://www.vmware.com/docs/vmware-vsan-technology-overview) | vSAN Overview |
| [vSAN Design Guide](https://www.vmware.com/docs/vmware-vsan-design-guide) | The Main vSAN Design Guide |
| [vSAN Strechted Cluster Guide](https://www.vmware.com/docs/vsan-stretched-cluster-guide) | The Main vSAN Strechted Cluster Guide |
| [vSAN Blog](https://blogs.vmware.com/cloud-foundation/technical/storage/) | The official vSAN Blog |
| [vSAN Space Efficiency](https://www.vmware.com/docs/vmw-vsan-space-efficiency) | Compression, Dedupe and Erasure Coding|

## NSX

| Name | Description |
| --- | --- |
| [Title](https://example.com) | Short description. |

## VCF (VMware Cloud Foundation)

| Name | Description |
| --- | --- |
| [Title](https://example.com) | Short description. |

## VKS (VMware vSphere Kubernetes Service)

| Name | Description |
| --- | --- |
| [Title](https://example.com) | Short description. |

## Aria

| Name | Description |
| --- | --- |
| [Title](https://example.com) | Short description. |

## KB & Community

| Name | Description |
| --- | --- |
| [Title](https://example.com) | Short description. |
