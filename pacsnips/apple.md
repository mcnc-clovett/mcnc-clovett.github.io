---
layout: page
title:  "Microsoft Services"
---
```js
/* Apple Services */
if (
    dnsDomainIs(host, ".apple.com") ||
    dnsDomainIs(host, ".icloud.com") ||
    dnsDomainIs(host, ".icloud-content.com") ||
    dnsDomainIs(host, "apple-finance.query.yahoo.com") ||
    dnsDomainIs(host, ".apple-cloudkit.com")
) return "DIRECT";
```
