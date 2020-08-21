---
layout: page
title:  "Google Services"
---
```js
/* most Google services */
if (
  dnsDomainIs(host, ".googleapis.com") ||
  dnsDomainIs(host, ".gstatic.com") ||
  dnsDomainIs(host, ".googleusercontent.com") ||
  dnsDomainIs(host, "clients1.google.com") ||
  dnsDomainIs(host, "clients2.google.com") ||
  dnsDomainIs(host, "clients3.google.com") ||
  dnsDomainIs(host, "clients4.google.com") ||
  dnsDomainIs(host, "clients5.google.com") ||
  dnsDomainIs(host, "clients6.google.com") ||
  dnsDomainIs(host, "accounts.google.com") ||
  dnsDomainIs(host, "tools.google.com") ||
  dnsDomainIs(host, "pack.google.com")
)
return "DIRECT";
```
```js
/* Google Classroom */
if (
  dnsDomainIs(host, "classroom.google.com") ||
  dnsDomainIs(host, "edu.google.com")
)
return "DIRECT";
```
```js
/* Chromebooks */
if (
  dnsDomainIs(host, "gweb-gettingstartedguide.appspot.com") ||
  dnsDomainIs(host, "omahaproxy.appspot.com")
)
return "DIRECT";
```
```js
/* Google Drive */
if (
  dnsDomainIs(host, "drive.google.com") ||
  dnsDomainIs(host, "googledrive.com")
)
return "DIRECT";
```
```js
/* Google Doc */
if (
  dnsDomainIs(host, "docs.google.com") ||
  dnsDomainIs(host, "googledocs.com")
)
return "DIRECT";
```
