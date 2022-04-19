---
layout: post
title:  "PAC File Syntax Explained"
date:   2020-12-16 00:00:00 -0400
categories: [PAC Files]
tags: [pacfiles, zscaler, filtering]
---
PAC files can seem a bit complicated at first, but keep in mind that they are just a javascript function with only two inputs. 
We'll be going over an example PAC file and discussing what each section does, as well as the correct syntax. Proper syntax 
is important, as making a slight mistake, like a misplaced semicolon, can cause the entire script to fail and 
**no traffic to be proxied**. For a full and comprehensive description of all available PAC file functions, check out 
[Mozilla's documentation](https://developer.mozilla.org/en-US/docs/Web/HTTP/Proxy_servers_and_tunneling/Proxy_Auto-Configuration_(PAC)_file){:target="_blank"}.

For the NCREN community that uses Zscaler, there is a "Verify" button that can be used to check the validity of your code. Keep 
in mind that this doesn't always catch problems in the syntax and you should **always** verify that your traffic is being proxied 
correctly after making a change.

The PAC file below is the example we'll be dicussing:

```js
function FindProxyForURL(url, host) {
    host = "."+host.toLowerCase();
    var privateIP = /^(0|10|127|192\.168|172\.1[6789]|172\.2[0-9]|172\.3[01]|169\.254|192\.88\.99)\.[0-9.]+$/;
    var resolved_ip = dnsResolve(host);
    
    /* If local server is resolvable to the correct address, go directly out */
    if (dnsResolve("internalserver.example.net") == "192.168.0.5" )
        return "DIRECT"; 

    /* Don't send non-FQDN or private IP auths to us */
    if (isPlainHostName(host) || isInNet(resolved_ip, "192.0.2.0","255.255.255.0") || privateIP.test(resolved_ip))
        return "DIRECT";

    /* Some example host bypasses */
    /* Single statement - complicated but faster on device */
    if (
        dnsDomainIs(host, ".example.com") ||
        dnsDomainIs(host, ".example.org") ||
        dnsDomainIs(host, "sub.example.net")
    ) return "DIRECT";
    /* Multiple statements  - easier but slower on device */
    if (dnsDomainIs(host, ".example.com")) return "DIRECT";
    if (dnsDomainIs(host, ".example.org")) return "DIRECT";
    if (dnsDomainIs(host, "sub.example.net")) return "DIRECT";
    
     /* Default Traffic Forwarding. */
    return "PROXY node1.example.com:9443; PROXY node2.example.com:9443";
}
```

---
### The Heading
```js
function FindProxyForURL(url, host) {
```
This first section is the start of the function that the browser runs when making a request for a web site. As you see below, 
there are two variables fed to the PAC file. They are `url` and `host`. As of Chrome 75 (and it's derivatives), these variables 
contain the following values given a URL such as `https://www.example.com/some/path/index.html`:
```
url  = https://www.example.com/
host = www.example.com
```
Notice there is no path information in the `url` variable. This is because 
[Google determined](https://chromium.googlesource.com/chromium/src/+/HEAD/net/docs/proxy.md#Arguments-passed-to-FindProxyForURL_in-PAC-scripts){:target="_blank"} this to be a security vulnerability and removed this information from PAC files. This means 
you are unable to use path information to direct traffic, and using paths will likely prevent your rules from working.

```js
host = "."+host.toLowerCase();
```
This line is modifying the `host` variable. The `host.toLowerCase()` portion converts the `host` to all lowercase characters 
to ensure consistent matching later on in the PAC file. In other words, `www.Example.com` will become `www.example.com`. 

Adding the `"."+` will prepend a dot to the `host`, which is a trick to prevent the need for entries for both `example.com` and `.example.com` in 
favor of just `.example.com`.

```js
var privateIP = /^(0|10|127|192\.168|172\.1[6789]|172\.2[0-9]|172\.3[01]|169\.254|192\.88\.99)\.[0-9.]+$/;
```
This line creates a variable called `privateIP` for several IP ranges that should not be proxied. These are `0.0.0.0`, `10.0.0.0/8`, 
`127.0.0.0/8`, `192.168.0.0/16`, `172.16.0.0/12`, `169.254.0.0/16` & `192.88.99.0/24`.
```js
var resolved_ip = dnsResolve(host);
```
This line creates a variable called `resolved_ip` that contains the IP address of the site you're trying to reach.

---
### Location Awareness
```js
/* If local server is resolvable to the correct address, go directly out */
if (dnsResolve("localsrv.ad.example.net") == "192.168.0.5" )
    return "DIRECT"; 
```
This block uses the `dnsResolve()` function to determine the IP address of `localsrv.ad.example.net` and find if it equals the 
provided value of `192.168.0.5`. If it does match, the traffic goes directly to the internet. This can be used with local DNS 
to test whether the client is on you network or not. If the client is off your network, naturally the hostanme will not resolve 
and the rest of the PAC file will be processed. It's best to put your location aware section above your exclusion list. This 
will avoid needlessly processing a long list of exclusions when the device is on-site.

---
### Send Local Traffic DIRECT
```js
/* Don't send non-FQDN or private IP auths to us */
if (isPlainHostName(host) || isInNet(resolved_ip, "192.0.2.0","255.255.255.0") || privateIP.test(resolved_ip))
    return "DIRECT";
```
This block tests for three different things, seperated by double pipes (`||`). The double pipe means `or`. `isPlainHostName(host)` 
tests if `host` is a simple hostname like `example-dc` or `localhost`. Next, `isInNet(resolved_ip, "192.0.2.0","255.255.255.0")` will 
test if the destination is in the 192.0.2.0/24 network. Lastly, `privateIP.test(resolved_ip)` tests whether the IP address of the 
destination matches the regular expression in the `privateIP` variable we dicussed earlier.

---
### Exclusions from PROXY
```js
/* Some example host bypasses */
/* Single statement - complicated but faster on device */
if (
    dnsDomainIs(host, ".example.com") ||
    dnsDomainIs(host, ".example.org") ||
    dnsDomainIs(host, "sub.example.net")
) return "DIRECT";
```
```js
/* Multiple statements  - easier but slower on device */
if (dnsDomainIs(host, ".example.com")) return "DIRECT";
if (dnsDomainIs(host, ".example.org")) return "DIRECT";
if (dnsDomainIs(host, "sub.example.net")) return "DIRECT";
```
Now for the section you'll have the most interaction with. This is where you place your rules for the various sites you wish not 
to be proxied. The above two blocks result in the **exact** same outcome. The difference is that the latter is more CPU intensive 
than the former. Its recommended to use whichever you are more comfortable with managing. 

In the first example, each test is separated by double pipes (`||`), which means `or`. The second exmaple places each test in its own 
statement on a seperate line.

The `dnsDomainIs()` function works by testing if the given string, such as `.example.com` is contained in the `host` variable. For 
example. If you visited `https://www.example.com`, the `host` variable would equal `www.example.com`. Your traffic would be sent directly 
to the internet because the `host` variable contains the string `.example.com`.

Sometimes a URL contains a simple domain name, such as `example.com`. This would not match `.example.com` and the traffic would be proxied. 
This can be resolved by adding an additional entry for the plain domain name. However, `dnsDomainIs(host, "example.com")` would also match 
`xxxexample.com`. As you can imagine, that would be a problem, so be careful using this method. A more efficient way would be to prepend the 
dot to every hostname, as we mentioned above, with `host = "."+host.toLowerCase();`. This allows you to always enter domains in the same 
fashion, such as `dnsDomainIs(host, ".example.com")`.

When matching a subdomain, you'll almost never want to provide a leading dot in your test. This is because doing so would require something 
preceeding it. For example:

```js
host = "sub.example.com"
dnsDomainIs(host, ".sub.example.com") // Does NOT match
dnsDomainIs(host, "sub.example.com")  // Does match
```
You would only want to add a leading dot if you expected something to always preceed it, such as `pre.sub.example.com`.

Remember, **never** add path information to `dnsDomainIs()`, or any other function that references the `host` variable. It will **never** 
result in the desired outcome. As mentioned earlier, as of Chrome version 75, path information isn't even included in the `url` variable.

---
### Send All Other Traffic to PROXY
```js
/* Default Traffic Forwarding. */
return "PROXY node1.example.com:9443; PROXY node2.example.com:9443";
```
This final block sends all traffic that wasn't matched above to the proxy server. These are prioritized from left to right. If the 
first server is unreachable, the second is used.
