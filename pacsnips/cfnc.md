---
layout: page
title:  "CFNC.org"
---
```js
/* CFNC */
if (
    /^(.+\.)?cfnc\.org$/.test(host) || // "cfnc.org" & ".cfnc.org"
    dnsDomainIs(host, "maxcdn.bootstrapcdn.com") ||
    dnsDomainIs(host, "cdnjs.cloudflare.com") ||
    dnsDomainIs(host, "code.jquery.com") ||
    dnsDomainIs(host, "use.fontawesome.com") ||
    dnsDomainIs(host, "api.humanesources.com") ||
    dnsDomainIs(host, ".ncresidency.org") ||
    dnsDomainIs(host, "studentaid.gov") ||
    dnsDomainIs(host, "fsaid.ed.gov")
) return "DIRECT";
```

You may not need all of the following Google exclusions depending on your current PAC file entries.
```js
/* CFNC Google Needs */
if (
    dnsDomainIs(host, "maps.googleapis.com") ||
    dnsDomainIs(host, "www.google.com") ||
    dnsDomainIs(host, "fonts.googleapis.com") ||
    dnsDomainIs(host, "ajax.googleapis.com") ||
    dnsDomainIs(host, "gstatic.com") ||
    dnsDomainIs(host, "cse.google.com") ||
    dnsDomainIs(host, "analytics.google.com") ||
    dnsDomainIs(host, "google-analytics.com") 
) return "DIRECT";
```