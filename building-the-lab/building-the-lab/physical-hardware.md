---
description: >-
  The proverbial "portal gun" allowing us access out of C-137 (or replacement)
  and enter the world of virtualization.
---

# Physical Hardware

![](../../.gitbook/assets/image%20%2818%29.png)

### Physical Server 

The budget friendly solution we both purchased was a dell R710.

 

![](../../.gitbook/assets/image%20%286%29.png)

 Some good sources to get one cheap

```bash
https://www.ebay.com/sch/i.html?_from=R40&_trksid=p2380057.m570.l1312.R2.TR0.TRC0.A0.H1.X.TRS2&_nkw=dell+r710&_sacat=0
https://gsaauctions.gov/
https://www.facebook.com/marketplace/
```

### Network Equipment

{% hint style="info" %}
Currently all networking equipment is consumer grade WiFi access points or mesh WiFi systems 
{% endhint %}

Currently looking into getting a couple L3 managed cisco switches and a decent cisco router. That way we can create the IPSEC tunnel via the hardware and also network more items in different enclaves. If we configure a network switch with a trunk and use explicit fail order on the distributed port group for the trunk you can access all the Vlans and use the pfsense as the router until you can afford a 2nd router. 

![This is the general principle described above \(except the wan stuff\) that will facilitate hardware in the loop.](../../.gitbook/assets/image%20%287%29.png)











