---
layout: post
title:  "Tracing Ports with Wireshark"
date:   2020-08-30 10:02:00 +0000
categories: wireshark
---
Tracing a port can be helpful when you need to make a configuration change for a certain 
drop connected to a switch. If the other end of the cable is connected to a switch that 
supports discovery protocols like CDP or LLDP, this can make life much easier. There is 
no need for a toner, or even an expensive cable testing device. You may not have been aware 
that Wireshark, the free packet sniffing tool, can give you all the information you need.

---

### Prerequisites

- You'll need Wireshark installed on your computer
    - [Download Wireshark here](https://www.wireshark.org/#download){:target="_blank"}
- If you'd like to follow along, download these example files for CDP and LLDP
    - [Example CDP Capture](https://gitlab.com/wireshark/wireshark/-/wikis/uploads/__moin_import__/attachments/SampleCaptures/cdp_v2_voice.pcap)
    - [Example LLDP Capture](https://gitlab.com/wireshark/wireshark/-/wikis/uploads/__moin_import__/attachments/SampleCaptures/lldp.detailed.pcap)

### Getting started

When you open Wireshark, select the interface that your are using to trace. Most likely, 
this will be the `Ethernet` interface. Next, copy and paste the following capture filter 
into the `Capture Filter` box as shown.

```
(ether host 01:00:0c:cc:cc:cc and ether[16:4] = 0x0300000C and ether[20:2] == 0x2000) or (ether proto 0x88cc)
```
![Capture Filter](/wireshark/cdp-lldp-filter.png)

Now start the capture and CDP and/or LLDP packets should begin to appear. It can take up to 
60 seconds to begin seeing packets depending on the timers on the remote device. For example, 
Cisco devices send out CDP packets every 60 seconds.

### Examining the Data

