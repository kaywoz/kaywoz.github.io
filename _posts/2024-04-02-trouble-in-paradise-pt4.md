---
title: trouble in paradise (pt4) - no opnsense gui for you, tailscale said.
date: 2024-04-02 13:01:00 +0100
categories: [homelab]
tags: [homelab,software,troubleshooting, tailscale, opnsense]     # TAG names should always be lowercase
comments: true
---

# Intro

Ok, so I finally got some time to (re)configure my firewall again, this time again with opnsense and vlans and crowdsec and yadayada. And again I ran into the issue that the opnsense management gui is not available over the tailnet.  Hmm...

# The problem.

I tried the regular things, checking the tailnet access permissions (since Im am going to have a fairly restrictive model) 
![tailscale access](/assets/images/2024-04-02-trouble-in-paradise-pt4/1.png)
 the tailscale interface firewall rules,
![firewall tailscale interface access](/assets/images/2024-04-02-trouble-in-paradise-pt4/2.png)
 that the tailscale interface is an allowed listener....
 ![tailscale interface set as listener](/assets/images/2024-04-02-trouble-in-paradise-pt4/3.png)

But to no avail. It still wasn't working. Some googling and dorking about later and i found this thread on [Reddit](https://www.reddit.com/r/OPNsenseFirewall/comments/11ww0sx/how_to_access_gui_via_tailscale_ip_address_on/)

According to the post, asymmetric routing is the culprit. M'ok, sounds plausible and it works for me by changing the CIDR to /10 and keeping the above as they are indeed necessary.

One might need to disable DNS rebinding checks/HTTP_REFERER checks as I've done in the pix or do some other magic if you like me, are NOT running split DNS. I run all(?) dns in cloudflare, even local homelab dns records. 

# What now?

All is good in tailscale land and I can keep the opnsense GUI off regular lan, most vlan etc. Feels nice doesn't it. Just need more time to start rolling out the main tailscale node filtering and I'm gonna be happy.



