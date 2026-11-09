# MISTAKES.md

A running log of mistakes made while writing or publishing posts in this repo: wrong assumptions, misread requirements, broken builds, anything kaywoz had to correct. Read this file in full before writing or changing a post. Add a new entry any time something turns out to have been based on a misunderstanding.

## Entry format

```markdown
### YYYY-MM-DD — <area/slug>

- **Trigger:** wrong assumption / build failed / misunderstood request / PR rejected
- **What happened:** one or two sentences, factual, no editorializing
- **Root cause:** why it happened
- **Fix applied:** what was changed to correct it
- **Rule going forward:** one concrete, checkable rule
```

Keep entries short and specific. The goal is a checklist that prevents repeat mistakes, not a diary.

---

### 2026-11-09 — posts (automated post disclaimer)

- **Trigger:** misunderstood request
- **What happened:** the first automated post (`2026-11-09-rclone-putio-on-a-headless-nas.md`) put the "Automated post" disclaimer box at the top, above the intro.
- **Root cause:** placement was chosen without asking kaywoz.
- **Fix applied:** disclaimer moved to the bottom of the post, after the sign-off, below a horizontal rule.
- **Rule going forward:** the automated post disclaimer (`{: .prompt-info }` box) always goes at the **bottom** of the post, never the top.

### 2026-11-09 — posts (tone)

- **Trigger:** misunderstood request
- **What happened:** the first automated post was written in a neutral, tutorial voice instead of kaywoz's own.
- **Root cause:** the existing posts weren't read before writing.
- **Fix applied:** post rewritten in kaywoz's voice as part of the "trouble in paradise" series.
- **Rule going forward:** read the existing `_posts/` before writing and match the voice: casual first person, lowercase titles, series titles where one fits (`trouble in paradise (ptN) - …`), `# Intro` / `# The problem.` / `# What now?` sections, short reactions ("Puh....", "Sigh...", "M'ok", "Lol"), rhetorical question-and-answer, funny image alt texts, sign-off like "Until then, cheers!" or "Aight peace!".

### 2026-11-09 — tabs (inline script)

- **Trigger:** broken page after deploy
- **What happened:** the weeklys script on the projects tab never ran ("Loading the latest weeklys..." stayed): `SyntaxError: Unexpected end of input`.
- **Root cause:** Chirpy compresses the HTML onto one line, so a `//` comment in the inline script commented out everything after it. The page wasn't checked live before handing over.
- **Fix applied:** `/* */` comment instead.
- **Rule going forward:** inline scripts in pages and posts use `/* */` comments only and end every statement with `;`; check them with `node --check` on a one-line copy before pushing.

### 2026-11-09 — tabs (d2 diagram size)

- **Trigger:** broken page after deploy
- **What happened:** the d2 diagram on the home0ps tab rendered at 0×0 and didn't show.
- **Root cause:** the d2 SVG has only a `viewBox`, no `width`/`height`, and sat lazy-loaded inside an inline link. Not checked live before handing over.
- **Fix applied:** `<img width="100%" style="width:100%;height:auto">` in a block-level link.
- **Rule going forward:** external SVGs get an explicit width; check every changed page on the live site after the deploy, before reporting it done.

