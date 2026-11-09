---
title: trouble in paradise (pt5) - put.io said no, four times
date: 2026-11-09 21:00:00 +0100
categories: [homelab]
tags: [homelab,software,troubleshooting,rclone,oauth,automated]     # TAG names should always be lowercase
image:
  path: /assets/images/2026-11-09-rclone-putio-on-a-headless-nas/1.png
comments: true
---

# Intro

So, new lab, new NAS, and of course I want rclone to talk to put.io from it. Easy right? Rclone does put.io, I've had put.io in the toolbox forever. Just run `rclone config`, click a button in a browser, done.

Except the NAS has no browser. Of course it doesn't. Puh....

Rclone has an answer for that though, `rclone authorize`. You run the OAuth dance on a machine that *does* have a browser, it spits out a token, and you paste that token into `rclone config` on the headless box. Sounds simple. It took four different errors to get there. Sigh...

# The problem(s).

## 1. "Invalid redirect_uri"

First try, put.io just refuses before I even get to log in.

```console
{"error_message":"Invalid redirect_uri","error_type":"INVALID_REDIRECT_URI","status_code":400}
```

M'ok. put.io only accepts the callback URL that's registered for the OAuth app making the request, and rclone *always* sends `http://localhost:53682/`. So two things need to line up:

* If you use your **own** put.io OAuth app (I do), its callback URL must be exactly `http://localhost:53682/`. Localhost, port 53682, trailing slash. No creativity.
* The browser machine has to sign in with **the same app** as the NAS config. `rclone authorize` takes the client ID and secret as extra arguments, and without them it falls back to rclone's built-in app. Which then won't refresh with your own app's secret on the NAS. Fun times later on.

```bash
read -rs S && rclone authorize "putio" <client_id> "$S" -vv; unset S
```

`read -s` keeps the secret off the screen and out of shell history, and `-vv` prints the actual sign-in URL so you can eyeball `client_id` and `redirect_uri`.

## 2. The button that just spins

Next round. Log in, click "Authorize application" and....

![spinny spinny spin](/assets/images/2026-11-09-rclone-putio-on-a-headless-nas/1.png)

Nothing. Forever. The kicker is that put.io *did* approve it, the authorization even showed up in my account. It redirected the browser to `http://localhost:53682/?state=…&code=…` and that request just never finished.

Is rclone listening? Yes. On `127.0.0.1`, and only there. Does the browser reach it through `localhost`? Apparently not. Lol.

Fix: grab the redirect URL from the browser dev tools (Network tab), swap `localhost` for `127.0.0.1` and open it while `rclone authorize` is still waiting. Done, code received.

## 3. "No code returned by remote server"

This one's on me (well, on the troubleshooting). To check if the auth server was even reachable, I opened `http://127.0.0.1:53682/` in another tab. Turns out rclone treats **any** request to that address as put.io's answer, and one without a code kills the sign-in with exactly this error ([`lib/oauthutil/oauthutil.go`](https://github.com/rclone/rclone/blob/master/lib/oauthutil/oauthutil.go) if you want to see it for yourself).

So yeah, don't poke port 53682 by hand while a sign-in is waiting. Lesson learned.

## 4. "Invalid value for key: 'client_secret'"

Redirect works, rclone gets the code, and then trading the code for a token fails.

```console
{"error_message":"Invalid value for key: 'client_secret'","error_type":"BadRequest","status_code":400}
```

Wrong secret in the hidden prompt. Thing is, put.io's app page shows both a **client secret** *and* a separate **OAuth token** for your own account, and rclone wants the client secret. Printing just the length before running is a cheap sanity check:

```bash
read -rs S && echo "secret length: ${#S}" && [ -n "$S" ] && rclone authorize "putio" <client_id> "$S" -vv; unset S
```

And.... token! Finally.

# Bonus round - the token is a password

put.io access tokens don't expire. Like, at all. So when `rclone authorize` prints the token it goes into the NAS's `config_token>` prompt and nowhere else, not chats, not tickets, not notes. Mine of course ended up in a chat once (facepalm), so I revoked that authorization in put.io and did the dance one more time. While you're in there, clean out the leftover authorizations from all the failed attempts.

Afterwards, check that the rclone config is mode 600. It now holds the client secret and the token (and in my case the crypt key for another remote too), so it belongs in the backups as well. Encrypted, obviously.

# What now?

Remote works, `rclone lsd` lists my stuff, all is good in put.io land. TL;DR for next time:

* The NAS config and `rclone authorize` must use the same client ID and secret.
* Your own app's callback URL must be exactly `http://localhost:53682/`.
* If the redirect hangs, use `127.0.0.1` instead of `localhost`.
* Never open port 53682 by hand while a sign-in is waiting.
* `-vv` is your friend.

Docs used: [rclone put.io](https://rclone.org/putio/) and [rclone remote setup](https://rclone.org/remote_setup/).

See you next time, same channel, different problem. Until then, cheers!

---

> Automated post: written by Claude from a troubleshooting session in my homelab, reviewed and merged by me.
{: .prompt-info }
