---
layout: page
title:  "General Backend Services"
---
```js
/* General Backend Services */
if (
  dnsDomainIs(host, ".akamaized.net") ||
  (host == "hcaptcha.com") ||
  dnsDomainIs(host, ".hcaptcha.com")
) return "DIRECT";
```
