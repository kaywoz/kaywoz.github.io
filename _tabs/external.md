---
layout: page
icon: fas fa-ethernet
order: 4
---

## 404: external services not found

```console
ka@lab:~$ curl -s http://services.lab/status
{"status":"rebuilding","eta":"soon(tm)","reason":"I rebuilt the lab. Again."}
ka@lab:~$ ping public-services
PING public-services: 56 data bytes
Request timeout for icmp_seq 0
Request timeout for icmp_seq 1
^C
--- public-services ping statistics ---
2 packets transmitted, 0 packets received, 100.0% packet loss
```

Nothing to see here yet. The public stuff went down with the old lab and the new one puts everything behind Tailscale first, so this page is currently a very well-guarded empty room.

Services will show up here again when (if?) something deserves to be public.

Until then, have a [Folding@home stat](https://folding.extremeoverclocking.com/user_summary.php?s=&u=378758) to look at, those GPUs still did some honest work.

![under construction, much wow](https://i.imgflip.com/1u2oyu.jpg)
