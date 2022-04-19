---
layout: page
title:  "Google Services"
---
```js
/* most Google services */
if (
  /clients\d\.google\.com$/.test(host) || // clients*.google.com
  dnsDomainIs(host, ".googleapis.com") ||
  dnsDomainIs(host, ".gstatic.com") ||
  dnsDomainIs(host, ".googleusercontent.com") ||
  dnsDomainIs(host, "accounts.google.com") ||
  dnsDomainIs(host, "meet.google.com") ||
  dnsDomainIs(host, "chat.google.com") ||
  dnsDomainIs(host, "tools.google.com") ||
  dnsDomainIs(host, "pack.google.com")
) return "DIRECT";
```
```js
/* YouTube */
if (
  dnsDomainIs(host, ".youtube.com") ||
  dnsDomainIs(host, ".youtube-nocookie.com") ||
  dnsDomainIs(host, ".ytimg.com") ||
  dnsDomainIs(host, ".googlevideo.com")
) return "DIRECT";
```
```js
/* Google Classroom */
if (
  dnsDomainIs(host, "classroom.google.com") ||
  dnsDomainIs(host, "edu.google.com")
) return "DIRECT";
```
```js
/* Chromebooks */
if (
  /\.gvt\d\.com$/.test(host) || // gvt*.com
  dnsDomainIs(host, "gweb-gettingstartedguide.appspot.com") ||
  dnsDomainIs(host, "omahaproxy.appspot.com")
) return "DIRECT";
```
```js
/* Google Drive */
if (
  dnsDomainIs(host, "drive.google.com") ||
  dnsDomainIs(host, "googledrive.com")
) return "DIRECT";
```
```js
/* Google Doc */
if (
  dnsDomainIs(host, "docs.google.com") ||
  dnsDomainIs(host, "googledocs.com")
) return "DIRECT";
```
