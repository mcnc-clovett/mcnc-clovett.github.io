---
layout: page
title:  "NCEdCloud"
---
```js
/* NCEdCloud */
if (
    dnsDomainIs(host, ".identitymgmt.net") ||
    dnsDomainIs(host, ".loom.com") ||
    dnsDomainIs(host, ".ncedcloud.org")
) return "DIRECT";
```