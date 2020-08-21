---
layout: page
title:  "Zoom"
---
```js
/* Zoom */
var zoom = /^zoom\.us(:443)?$/;
if (
    zoom.test(host) ||
    dnsDomainIs(host, ".zoom.us") ||
    dnsDomainIs(host, ".cloudfront.net") ||
    dnsDomainIs(host, ".ada.support")
) return "DIRECT";
```
