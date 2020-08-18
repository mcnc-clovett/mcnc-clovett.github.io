---
layout: post
title:  "PAC File Code for Various Applications"
date:   2020-08-18 14:47:00 +0000
categories: zscaler
---

This page hosts numerous PAC file entries that we have gathered over 
time. Use your browser's search function to find the service you may
be looking.

## Google Services
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
## Microsoft Services
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
## GoGuardian
```js
/* Added for GoGuardian */
if (
  dnsDomainIs(host, ".goguardain.com") ||
  dnsDomainIs(host, ".laptoplookout.com") ||
  dnsDomainIs(host, "kinesis.us-west-2.amazonaws.com") ||
  dnsDomainIs(host, "hosted-extensions.s3.us-west-2.amazonaws.com") ||
  dnsDomainIs(host, "x3-report-uploads.s3.us-west-2.amazonaws.com") ||
  dnsDomainIs(host, "beacon-report-uploads-prod.s3.us-west-2.amazonaws.com")
)
return "DIRECT";
```

