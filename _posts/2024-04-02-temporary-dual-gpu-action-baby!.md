---
title: temporary dual gpu action baby!
date: 2024-04-02 13:01:00 +0100
categories: [homelab]
tags: [homelab,hardware,software,folding,supercomputing,distributed computing]     # TAG names should always be lowercase
comments: true
---

# Intro

Moved over from my dedicated [Hostkey](https://hostkey.com/gpu-dedicated-servers/) 5950x+RTX4090 setup to a more modest vm with passthrough GPU, still an RTX4090. Setup was fairly easy and sweet, even setup my healthchecks.io scripts to monitor the GPU-worker output and it has been stable so far. Still, the 40M PPD should make you go all jolly inside when you see the below!

![image tooltip here](/assets/images/2024-04-02-temporary-dual-gpu-action-baby!/fahh.png)

# What now?

So, #2 spot is ~3 weeks away and then the lucrative #1 is a mere ~1.5 months away. Am starting to look into some sort of orchestration for dual tasking the GPU with 80% GPU time for FaH and 20% for some hashcat play for hashmob.net [Hostkey](https://hashmob.net/). We will see.

Aight peace!