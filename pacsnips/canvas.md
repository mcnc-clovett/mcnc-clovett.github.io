---
layout: page
title:  "Canvas"
---
```js
/* Canvas */
if (
  dnsDomainIs(host, ".inscloudgate.net") ||
  dnsDomainIs(host, ".canvaslms.com") ||
  dnsDomainIs(host, ".canvas-user-content.com") ||
  dnsDomainIs(host, ".instructure.com") ||
  dnsDomainIs(host, ".ncedcloud.com") ||
  dnsDomainIs(host, ".ncpublicschools.gov")
) return "DIRECT";
```
