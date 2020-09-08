---
layout: page
title:  "Wireshark Profiles"
---

---
## Zscaler PAC File Troubleshooting

[PAC_Troubleshooting.zip](./profiles/PAC_Troubleshooting.zip)

Having a HAR file is great when you're in a web browser. Sometimes you need to troubleshoot apps 
that don't give you that convenience. This profile will help with that problem by posting the hosts 
needed in your pac file directly to the screen. Once the profile is imported and loaded, set the 
capture filter to the saved `Zscaler ports: port 80 or port 9443` on your internet connected interface.
Three display filters have been saved, `PAC Files`, `Hosts` and `PAC n Hosts`. These will display 
the PAC files as they are downloaded and the "host" requests as they are made respectively. The entries in the `Host` 
column are what you need to add to your PAC file.

## Port Tracing with CDP

[Port_Discovery.zip](./profiles/Port_Discovery.zip)

No expensive port tracing tools needed, just your laptop and a CAT5/6 cable. Once you have the profile 
imported and loaded, start your capture with the saved `CDP` capture filter on your `Ethernet` interface. 
Columns are pre-set for all the data you'll need for the port you're connected to.
