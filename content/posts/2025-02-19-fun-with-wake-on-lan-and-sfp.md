---
title: Fun with Wake on LAN and SFP+
date: 2025-02-19T16:26:00+01:00
draft: false
---
As I have some extra time available at the moment, I'm currently working down on my _Annoying Technical Issues But Not Important Enough To Fix Right Away_ list. And one of the tasks is to fix my Wake on LAN setup.

As a freelancer, I usually work out of my office on my workstation, but sometimes I'm onside at customers' with a notebook. As most of the things I do are on my Workstation, access to that very thing is often quite import. Yes, source code is on some Git server, most of my data is on my NAS, but special stuff, like a virtual machine might only exist on my Workstation. Especially when I'm abroad or out for multiple days, I don't want my PC running for 24/7, that would just be a waste of energy. However, I just might need to access it. So Wake on LAN it is. VPN home, send the magic package and RDP to my PC.

I did my Wake on LAN setup a couple of years ago. It was a big hassle to make it work, because you need to tweak UEFI, network card, Windows energy saving and so on. But eventually it worked, even thought only with my second `1G` Intel network card, and not with my main `10G` Marvell one. Yes I know, completely overkill, who needs a `10G` network card? But here we are.

At some point, it suddenly didn't work anymore. As I knew how much time it probably would take me to fix it, I did the only logical thing: I procrastinated this task as long as I could.

Today was finally the day to investigate it. It was exactly the kind of debugging I feared: randomly tweaking settings, updating drivers etc. till it worked again with the `1G` Intel card. It was missing the `Enable PME` setting (probably). I could have sworn, that my network adapter didn't show me that option at the beginning, so either also drivers needed an update or I was just blind. All in all, it took me about two hours to fix it.

### Curiosity always wins

However, now I was confident and curious: can I also make it work with the `10G` Marvell network card? While having two network cables to my workstation is no biggie, it is still quite unnecessary (yes I'm still using my `10G` Marvell card mainly - bite me). After more debugging, switching cables and ports etc. I realized: the network card is not the problem, it works just fine. However, the network port on my router seems to be the problem.

You see, my router, an Unifi Dream Machine Pro, doesn't have a native `10G` `RJ45` port, but it has a `SFP+` module slot. Thanks to [this thread](https://community.ui.com/questions/WoL-packets-blocked-by-SFP-port-on-a-US-8-150W/dd5e9ed2-a3da-488a-93fb-4f8817bd0510), which gave the correct hint that the module is the problem: my `SFP+` to `RJ45` module does not support Fast Ethernet, which is used when the system is in S5 mode (shutdown). So the uplink - at least from the point of view of the router - was down. A particularly misleading detail: the LEDs on the network card were still on, so I just _assumed_, that there would be an uplink.

### The solution now?

Well, it looks like I'll have to stick with two cables. I would either need a `SFP+` module, that also supports Fast Ethernet (and I'm not even sure if something like that exists). Or I would need a proper switch with a normal `10G` `RJ45` port, that would support Fast Ethernet. Both options would not justify the cost/benefit factor for the fun of having a `10G` uplink into your local network. At least I know the reason now and can make the second cable at my desk permanently. Sounds like another TODO for the _Annoying Technical Issues But Not Important Enough To Fix Right Away_ list.

### Why even the `10G` thing?

Good question! My NAS has 4x `1G` uplink - at least in theory. My thought was, maybe I could do some link aggregation and get more than one gigabit per second of speed out of it. The problem is: the Unifi Dream Machine Pro doesn't support link aggregation, a small detail I missed. No worries, I have a NETGEAR switch, that supports link aggregation and another SFP+ port available on my router. Well turns out the NETGEAR switch only has `SFP` and not `SFP+`, therefore it could only support `1GB` again. So till the day I invest into new hardware, no real use for the `10G` uplink of my PC, unfortunately. But I paid for it, so I will use it.
