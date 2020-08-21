---
layout: page
title:  "Microsoft Services"
---
```js
/* Microsoft */
if (
  dnsDomainIs(host, ".microsoftonline-p.com") ||
  dnsDomainIs(host, ".microsoftonline-p.net") ||
  dnsDomainIs(host, ".microsoftonline.com") ||
  dnsDomainIs(host, ".microsoftonlineimages.com") ||
  dnsDomainIs(host, ".microsoftonlinesupport.net") ||
  dnsDomainIs(host, ".msecnd.net") ||
  dnsDomainIs(host, ".office.com") ||
  dnsDomainIs(host, ".msauth.net") ||
  dnsDomainIs(host, ".msftauth.net") ||
  dnsDomainIs(host, ".officehome.msocdn.com") ||
  dnsDomainIs(host, ".msocdn.com") ||
  dnsDomainIs(host, ".office.net") ||
  dnsDomainIs(host, ".office365.com") ||
  dnsDomainIs(host, ".officeapps.live.com") ||
  dnsDomainIs(host, ".outlook.com") ||
  dnsDomainIs(host, "login.microsoftonline.com") ||
  dnsDomainIs(host, "onedrive.live.com")
)
return "DIRECT";
```
