---
layout: page
title:  "Update Services"
---
```js
/* Update Services */
if (
  /\.?download\.windowsupdate\.com$/.test(host) ||
  /\.?tlu\.dl\.delivery\.mp\.microsoft\.com$/.test(host) ||
  dnsDomainIs(host, "officecdn.microsoft.com") ||
  dnsDomainIs(host, "officecdn.microsoft.com.edgesuite.net") ||
  dnsDomainIs(host, "dl.google.com") ||
  dnsDomainIs(host, ".gvt1.com") ||
  dnsDomainIs(host, "ardownload.adobe.com") ||
  dnsDomainIs(host, "ccmdl.adobe.com") ||
  dnsDomainIs(host, "agsupdate.adobe.com")
) return "DIRECT";
```
