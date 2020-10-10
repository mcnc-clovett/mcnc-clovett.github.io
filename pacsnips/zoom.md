---
layout: page
title:  "Zoom"
---
```js
/* Zoom */
if (
    /^(.+\.)?zoom\.us$/.test(host) ||
    dnsDomainIs(host, ".zoom.us") ||
    dnsDomainIs(host, ".cloudfront.net") ||
    dnsDomainIs(host, ".ada.support")
) return "DIRECT";
```
