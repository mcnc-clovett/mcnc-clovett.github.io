---
layout: page
title:  "Minecraft for Education"
---
```js
/* Minecraft for Education */
if (
    dnsDomainIs(host, ".minecrafteduservices.com") ||
    dnsDomainIs(host, ".xboxlive.com") ||
    dnsDomainIs(host, ".playfabapi.com") ||
    dnsDomainIs(host, "education.minecraft.net") ||
    dnsDomainIs(host, "minecraftprod.rtep.msgamestudios.com") ||
    dnsDomainIs(host, "contentstorage.onenote.office.net") ||
    dnsDomainIs(host, "meedownloads.azureedge.net") ||
    dnsDomainIs(host, "meedownloads.blob.core.windows.net") ||
    dnsDomainIs(host, "cognitiveservices.azure.com") ||
    dnsDomainIs(host, "learningtools.onenote.com") ||
    dnsDomainIs(host, "minecraft.makecode.com") ||
    dnsDomainIs(host, "makecode.com") ||
    dnsDomainIs(host, "trg-minecraft.userpxt.io") ||
    dnsDomainIs(host, "pxt.azureedge.net") ||
    dnsDomainIs(host, "api.github.com") ||
    dnsDomainIs(host, "www.tynker.com") ||
    dnsDomainIs(host, ".msftauth.net") ||
    dnsDomainIs(host, "login.live.com") ||
    dnsDomainIs(host, "login.microsoftonline.com")
)
return "DIRECT";
```