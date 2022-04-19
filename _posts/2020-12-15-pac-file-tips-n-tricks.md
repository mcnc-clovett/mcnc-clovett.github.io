---
layout: post
title:  "PAC File Tips & Tricks"
date:   2020-12-15 00:00:00 -0400
categories: [PAC Files]
tags: [pacfiles, zscaler, filtering]
---
Lets try to clear up some common questions and issues with PAC files

---

## Understand what the "url" and "host" variables are

You will always see the following on the top line of a PAC file

```js
function FindProxyForURL(url, host) {
```

Most schools districts are using Chrome, so we'll cover what Chrome feeds to the `url` and `host` variables.
Using the example of `https://www.example.com/some/path`, these two variables will hold the following values:

```
url  = https://www.example.com/
host = www.example.com
```
Notice the path is not included in either variable. This is due to 
[Google making a decision](https://chromium.googlesource.com/chromium/src/+/HEAD/net/docs/proxy.md#Arguments-passed-to-FindProxyForURL_in-PAC-scripts){:target="_blank"} to remove this path information due to "security concerns." Using URL path information in a PAC 
file will have negative results on Chrome clients.

---
## Dots are NOT wildcard characters

In Zscaler, preceeding dots such as the one in `.example.com`, are typically used as wildcard characters. In PAC files, this isn't the case. Dots are simply dots, therefore `.example.com` does NOT match `example.com`. It does match `www.example.com`, however. So here are some things to note:

### When matching a subdomain

Typically, when you are matching a subdomain, you want to match it specifically. When matching `subdomain.example.com`
you'll use the following:

```js
if (dnsDomainIs(host, "subdomain.example.com")) return "DIRECT"; // good example
```

If you add a preceeding dot to this hostname, it will not match due to the full `.subdomain.example.com` string not being in 
the `host` variable. For example, the following will **not** match `subdomain.example.com`:

```js
if (dnsDomainIs(host, ".subdomain.example.com")) return "DIRECT"; //bad example
```

### When matching a domain without a subdomain

When matching a domain that has no subdomain, you'll have to leave off the preceeding dot, like so:

```js
if (dnsDomainIs(host, "example.com")) return "DIRECT";
```

Keep in mind that this may open up problems, such as a site like `xxxexample.com`, which would definitely be a problem. 
Technically, this entry would also match `www.example.com`.

### How to eliminate the "dot, no dot" issue

If you add a preceeding dot to the `host` variable at the top of the PAC file, you will only need the `.example.com` entries.
For example:

```js
function FindProxyForURL(url, host) {
    host = "."+host.toLowerCase()
```

This will make the host variable for `www.example.com` & `example.com` look like the following respectively:

```
host = .www.example.com
host = .example.com
```

So as you can see, the following rule would match either example:

```js
if (dnsDomainIs(host, ".example.com")) return "DIRECT";
```

And this rule would still match the subdomain rule, if you need to be specific:

```js
if (dnsDomainIs(host, "www.example.com")) return "DIRECT";
```

#### Alternative resolution to the "no dot" issue

You can also use regular expressions, also known as regex, to match domains without matching potentially bad sites. 
This is only recommended if you have indepth knowledge of regex. For example to match `example.com` and not `xxxexample.com`,
you would use the following:

```js
if (/^(.+\.)?example\.com$/.test(host)) return "DIRECT";
```