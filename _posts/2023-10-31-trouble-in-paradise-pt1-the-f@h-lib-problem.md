---
title: trouble in paradise (pt1) - the f@h lib problem
date: 2023-10-31 08:01:00 +0100
categories: [homelab]
tags: [homelab,hardware,software,folding,supercomputing,distributed computing,troubleshooting]     # TAG names should always be lowercase
comments: true
---

# Intro

So many friggin problems running F@h, everything from Ubuntu auto-updating the Nvidia drivers, to regular power outages and work servers turning up unavailable. Puh....

Of course I randomly checked it today and it turns out it's been missing work for the past 24h, (I really really need to hook up some sort of scripted monitor looking for strings in the fahlog.txt or maybe just change detection on a F@h scoring site?)

# The problem.

Apparently Ubuntu did not want to recognize .so-file in the file system any longer, why? I dunno...

```console
[...]libOpenCL.so.1: cannot open shared object file: No such file or directory
```

![image tooltip here](/assets/images/2023-10-31-some-fah-troubleshooting-(again)/so_fail.PNG)

Not much to go on, is the file there? Yes. Is it valid? Probably. Did I change anything? Nope.

Sigh...

Some 5 minute googling heroics later I only needed to update apt and reinstall some-something.

```bash
sudo apt update
sudo apt install ocl-icd-opencl-dev
```

# What now?

Well, looks like we are happy again!

![image tooltip here](/assets/images/2023-10-31-some-fah-troubleshooting-(again)/fah_happy.PNG)

Really really happy!

![image tooltip here](/assets/images/2023-10-31-some-fah-troubleshooting-(again)/fah_gpustat.PNG)

But I truly need to look into monitoring F@h jobs, maybe via healtchecks.io or similar. Anyway, until then, cheers!