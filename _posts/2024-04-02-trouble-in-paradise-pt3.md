---
title: trouble in paradise (pt3) - the missing person((al)token)
date: 2024-04-02 13:01:00 +0100
categories: [wrk]
tags: [wrk,software,troubleshooting]     # TAG names should always be lowercase
comments: true
---

# Intro

As I'm between consultancy gigs, I'm currently tasked with random internal stuff with the company. Have taken it upon myself to improve (make it work?) an old forensics package collector script, but has been kicking my a$$.

Was asked to look into why the internal jira bot no longer messaged personnel changes to slack and no, this is nothing I deal with regularly so my imposter syndrome kicked in hard.

# The problem.

Yeah, obvious problem. If you know where to look in GCP, which used temporary runners to the python code for the bot, lol.

![obvious problem is obvious](/assets/images/2024-04-02-trouble-in-paradise-pt3/1.png)

Something something is requesting something with the key OOTA and "got 0 results", ok are there results? is this expected?

Im guessing here that the second event cluster is the giveaway; "expected only 1 project, found []" 

So it got back an empty array, so the runner cannot read the project properties.... Sounds like.... permissions

A quick check later, (LONG CHECK, so confusing reading other peoples documentation sometimes)  and we can start to check the tokens used.

![one is a token, the other is... also a token](/assets/images/2024-04-02-trouble-in-paradise-pt3/2.png)

Yes, so first I thought that the token maybe had expired (eventhough the state was valid according to the GCP console). But I luckily enough had a look at the old token before swapping them around (as all the token values are visible, did not expect that) and noticed that...it was set with an email address for the user's old email but not the new email address post name change.

![why can i view the token values?](/assets/images/2024-04-02-trouble-in-paradise-pt3/3.png)

 Wow. This again. So many bad memories of maternities and weddings fuxxing up scheduled jobs across enterprises. 

Anyway, it was as easy as changing the email in the token to the current one, so that it mapped correctly when authenticating...

https://imgflip.com/i/8latcx

# What now?

Well, as the old adage says, trust that a namechange changes more than you think, or something. Lol.

Also, have change processes for this kind of thing, and thanks for people making reasonable documentation. Aight peace.



