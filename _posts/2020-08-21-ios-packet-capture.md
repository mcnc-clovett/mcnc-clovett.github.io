---
layout: post
title:  "Capture Packets on iOS Devices"
date:   2020-08-21 01:11:00 -0400
categories: [Networking, Wireshark]
tags: [networking, wireshark, apple, ios, troubleshooting]
---
From time to time you may need to gather some information about what an iOS device is doing on 
your network. For instance, you may need to know what URLs an app is using so it can be bypassed 
in your content filter.

---

#### **Before you begin, you need:**
- MacBook with [Wireshark](https://www.wireshark.org/#download){:target="_blank"} & [Xcode](https://apps.apple.com/us/app/xcode/id497799835){:target="_blank"} installed
- iOS device with cable to attach to the MacBook

You'll need the UDID of you iOS device, so plug it into you MacBook and open finder. You should 
see the device in the left-hand pane. Click on it, then click on the device's description at the top.

![iPadInfo](/images/wireshark/ipadinfo.png)

This will display the serial number and UDID. Right click the UDID and click `Copy UDID`.

![iPadInfo](/images/wireshark/ipadudid.png)

Now that you have the UDID, open the Terminal app and use this command to create the interface we'll use to capture packets.

```console
$ rvictl -s PASTE-YOUR-UDID-HERE
```

If successful, you should see your new interface on screen. In most instances, it will be `rvi0` as seen below.

```console
Starting device 00000000-0000000000000000 [SUCCEEDED] with interface rvi0
```

Now open Wireshark, and you should see this new interface listed as a capture option.

![iPadInfo](/images/wireshark/ipadcaptureint.png)

You can now proceed as you normally would with the capture filter of your choosing.