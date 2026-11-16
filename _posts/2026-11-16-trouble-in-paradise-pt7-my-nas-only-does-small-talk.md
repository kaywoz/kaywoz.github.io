---
title: trouble in paradise (pt7) - my nas only does small talk
date: 2026-11-16 12:00:00 +0100
categories: [homelab]
tags: [homelab,troubleshooting,networking,nas,realtek,mos,automated]     # TAG names should always be lowercase
comments: true
---

# Intro

New toy on the desk: a 4-bay all-NVMe NAS with **5GbE** onboard. The plan was to replace my current storage box, so I wrote a proper weekend test plan. Disk speeds, thermals, network, the works. Boring stuff, ticking boxes, Saturday done.

Disks? Fine. CPU under 45 minutes of stress? 74°C, zero throttling. Network, single stream? 2.35 Gbit/s to my 2.5G test box, zero retransmits.

Then I ran iperf3 with **four** streams.

```console
0s 0.63 Gbit/s retr=5133
1s 0 Gbit/s retr=4
2s 0 Gbit/s retr=0
...
28s 0 Gbit/s retr=0
```

M'ok.

{% comment %}
image goes here, e.g.:
![a nas sitting in a corner, refusing to talk](/assets/images/2026-11-16-trouble-in-paradise-pt7-my-nas-only-does-small-talk/silence.png){: w="600" }
_The NAS, after four TCP streams._
{% endcomment %}

# The problem.

One burst of packets, a few thousand retransmits, then **nothing**. For 28 seconds. Then the box "hung": SSH dead, MOS UI dead, the KVM console black. And the next time I looked, it had rebooted. Spooky!

(It had not rebooted itself. That was me. I forgot. Lol.)

## The red herrings

What followed was a weekend of very confident theories:

* **"The library keeps disappearing."** True, but unrelated. MOS runs from RAM, and its `apt-get` is a wrapper that brings back the packages you named at boot, but not their dependencies. So `iperf3` does not survive a reboot, nor `libiperf0`. Fun, but not it.
* **"The NIC changed its name."** Also true! `eth0` when the USB 5G adapter is unplugged, `eth1` when it's plugged in. Also not it.
* **"It's a hard hang."** Ctrl-Alt-Del worked and the box shut down cleanly. Not a hard hang.
* **"It's jumbo frames."** MTU 1500.
* **"It's offloads."** I ran an A/B test with TSO/GSO off, everything off, everything on. All three failed. Sigh...
* **"It's PCIe power saving."** ASPM was already off.
* **"It's the firewall dropping INVALID packets."** No such rules on either end.
* **"It's out of memory."** 5.4 GB free.
* **"Just turn on flow control."** The switch has it on. The NIC (RTL8126A on the in-kernel `r8169` driver) just... doesn't negotiate it, and `ethtool -A` changes nothing.

Every theory came with a test, and every test said no. Puh....

## The KVM tax

Side quest: my IP-KVM translates my keyboard layout. So `;`, `|`, `$` and `&` don't arrive as typed. Ever tried to debug Linux without semicolons and pipes? One command per line, like it's 1983.

## The clue

Then this happened in the "dead" state:

```console
$ ssh -vvv root@<nas>
debug1: Remote protocol version 2.0, remote software version OpenSSH_10.0p2 Debian
debug1: SSH2_MSG_KEXINIT sent
```

...and then nothing. Wait. The NAS **answered**. It sent its banner, a tiny packet. It just never got through the key exchange, where packets are about a kilobyte or more.

So, ping with sizes:

```console
  100 bytes: 2 packets received
  500 bytes: 0 packets received
 1000 bytes: 0 packets received
 1400 bytes: 0 packets received
```

There it is. **The NAS only does small talk.** Small frames in, fine. Anything bigger, gone. And the RX counters proved the big frames never even reached the OS. Zero errors, zero drops, zero missed. The NIC just quietly threw them away.

One `ip link set eth0 down` / `up` later, everything worked again. So the box was never hung. It just couldn't hear anyone who talked in full sentences.

## The trigger

Two more tests to pin it down:

| Setup | 4 streams, NAS sending |
|---|---|
| Through the switch, NAS at 5G, client at 2.5G | ❌ stalls, 6 out of 6 times |
| Direct cable, no switch | ✅ 2.34 Gbit/s, 0 retransmits |
| Through the switch, NAS forced to 2.5G | ✅ 2.34 Gbit/s, 0 retransmits |

So here's the chain:

1. The NAS sends at 5G into a 2.5G client.
2. The switch has to step down, and with no working flow control on the NAS side, it drops packets in bursts. Normal switch behaviour.
3. After those bursts, the NIC/driver gets stuck and drops every received frame over a few hundred bytes, until someone resets the interface. **That's the bug.** Loss should never wedge a NIC.

# What now?

The "fix" would be forcing the port to 2.5G. But I didn't buy a 5G box to run it at 2.5G. So instead it gets a watchdog that does the reset for me:

* Every minute, ping the gateway small and large.
* Small answers but large doesn't, twice in a row? That's the wedge. Bounce the interface, put the default route back (a link down eats it), check again.
* Both fail? That's a different problem, so it doesn't touch anything.
* At most 4 bounces an hour, with a cooldown if a bounce didn't help. Every bounce gets logged on the boot stick, so I'll know how often this actually happens in real life.

It's in my homelab repo as [`nic-watchdog.sh`](https://github.com/kaywoz/home0ps/blob/main/config-files/mos/nic-watchdog.sh). A minute-or-so hiccup instead of a "drive to the box with a keyboard" outage. Homelab-grade. Good enough until there's a real driver fix: Realtek's own `r8126` or a newer kernel. That's next.

TL;DR for next time:

* "Ping works but nothing else does" → **ping with sizes** before anything else. `ping -s 100` vs `ping -s 1000`.
* `ssh -vvv` stopping at `KEXINIT` means small packets get through and big ones don't.
* Check the RX counters during a test. They tell you which direction is broken.
* A direct cable is the fastest way to take the switch out of the equation.
* Write down when *you* reboot the box. Lol.

Until then, cheers!

---

> Automated post: written by Claude from our troubleshooting session and the test log, reviewed and merged by me. Some of the confident wrong theories were Claude's.
{: .prompt-info }
