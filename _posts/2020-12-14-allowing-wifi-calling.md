---
layout: post
title:  "Allowing Wifi Calling"
date:   2020-12-14 00:00:00 -0400
categories: [Networking, Firewall]
tags: [networking, wifi calling, cell phone]
---
Cell phone coverage can be spotty in some areas, as we all know. This is especially true in old concrete-walled buildings. 
Since most, if not all, educational institutions today have a wireless infrastructure, we have a way of mitigating this problem. 
Through a feature called wifi calling, we can allow users to make and receive phone calls using the local internet infrastructure. 
This is especially important for possible emergency situations where someone may not have immediate access to a landline.

There is a slight problem with this, however. Allowing wifi calling requires allowing `UDP` ports `500` & `4500` out of your network. 
If you don't recognize these ports, they are standard ports for VPN protocols. Yes, those pesky applications that allow students 
to bypass your filtering system. Below, we'll discuss what exactly is needed to allow wifi calling **without** allowing VPN applications.

---

### What are the destinations
We have to determine where the traffic is going so that we can limit where the needed ports can go. Below are the known hostnames for the 
major carriers:

|    Verizon   |       AT&T       |    T-Mobile   |    US Cellular   |
|:------------:|:----------------:|:-------------:|:----------------:|
| wo.vzwwo.com | epdg.epc.att.net | 208.54.0.0/17 | 198.230.224.0/20 |

### What rules are needed in the firewall
Now that we have the destinations, and we know the ports are `UDP` `500` & `4500`, we can add rules to the firewall to allow these 
connections out. Before you begin, its important that your firewall can resolve hostnames. If you have access to your firewall, try pinging 
one of the above addresses from the firewall itself. If the hostname resolves, you may continue, otherwise you'll need to address this issue.

Many entities are using Cisco ASAs currently, so the needed code for these firewalls is as follows:

```console
object network Wifi_Calling_ATT
 fqdn v4 epdg.epc.att.net
object network Wifi_Calling_Verizon
 fqdn v4 wo.vzwwo.com
object network Wifi_Calling_TMobile
 subnet 208.54.0.0 255.255.128.0
object network Wifi_Calling_USCellular
 subnet 198.230.224.0 255.255.240.0
object-group network Wifi_Calling_Hosts
 network-object object Wifi_Calling_ATT
 network-object object Wifi_Calling_Verizon
 network-object object Wifi_Calling_TMobile
 network-object object Wifi_Calling_USCellular
object-group service Wifi_Calling_Svc udp
 port-object eq isakmp
 port-object eq 4500
access-list inside_access_in extended permit udp any object-group Wifi_Calling_Hosts object-group Wifi_Calling_Svc
```

Essentially, you need to create a `UDP` service/port group and add port `500` (also known as `isakmp`) as well as `4500` to it. Also, create 
objects for the Wifi Calling destinations in the table above and add them to a group. You can then add a rule on your inside interface, allowing 
traffic out on these ports to the needed destinations. As long as you are not allowing these ports to flow in another rule, VPN should remain 
blocked and Wifi Calling should now be usable.

You may want to add a source subnet for where you want Wifi calls to be made from, such as a certain Wifi network where cell phones will be 
connecting. A typical Wifi call will use less than 250Kbps, most times much less. Bandwidth shouldn't be a concern when deciding whether to 
allow Wifi calling on your network.
