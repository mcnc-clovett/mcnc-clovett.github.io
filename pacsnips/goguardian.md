---
layout: page
title:  "GoGuardian"
---
```js
/* Added for GoGuardian */
if (
  dnsDomainIs(host, ".goguardain.com") ||
  dnsDomainIs(host, ".laptoplookout.com") ||
  dnsDomainIs(host, "kinesis.us-west-2.amazonaws.com") ||
  dnsDomainIs(host, "hosted-extensions.s3.us-west-2.amazonaws.com") ||
  dnsDomainIs(host, "x3-report-uploads.s3.us-west-2.amazonaws.com") ||
  dnsDomainIs(host, "beacon-report-uploads-prod.s3.us-west-2.amazonaws.com")
) return "DIRECT";
```
