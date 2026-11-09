---
# the default layout is 'page'
icon: fas fa-tasks
order: 3
---

This page catalogues some of my projects. Many of these projects pages are updated as new development occurs, and include a wrap-up of all of the corresponding blog posts. Enjoy!


## placeholders below 

- [ ] projects
  - [x] making decisions on what comes first
  - [ ] the great DNS trip
  - [ ] the neverending storage decision
  - [ ] automate yourself and wreck yourself
  - [ ] the real REAL lab
  - [ ] the home server that could but was not omv or truenas

## The current state of tasks in projects @h0me0ps can be found below.

What's being worked on right now lives on the [home0ps project board](https://github.com/users/kaywoz/projects/4) (backlog, selected, done).

Every Monday a bot summarizes the week in home0ps and refreshes the README. The latest weeklys, straight from GitHub:

<div id="weeklies"><p>Loading the latest weeklys...</p></div>

<p><a href="https://github.com/kaywoz/home0ps/pulls?q=is%3Apr+is%3Amerged+in%3Atitle+%22weekly+README+refresh%22">All weeklys on GitHub</a></p>

<script>
(async () => {
  const el = document.getElementById('weeklies');
  const all = 'https://github.com/kaywoz/home0ps/pulls?q=is%3Apr+is%3Amerged+in%3Atitle+%22weekly+README+refresh%22';
  try {
    const q = encodeURIComponent('repo:kaywoz/home0ps is:pr is:merged in:title "weekly README refresh"');
    const r = await fetch('https://api.github.com/search/issues?q=' + q + '&sort=created&order=desc&per_page=4',
      { headers: { Accept: 'application/vnd.github.html+json' } });
    if (!r.ok) throw new Error('GitHub API ' + r.status);
    const { items } = await r.json();
    if (!items || !items.length) throw new Error('no weeklys found');
    el.textContent = '';
    items.forEach((it, i) => {
      const d = document.createElement('details');
      if (i === 0) d.open = true;
      const s = document.createElement('summary');
      const a = document.createElement('a');
      a.href = it.html_url;
      a.textContent = (it.title.match(/\((\d{4}-\d{2}-\d{2})\)/) || [, it.title])[1];
      s.append('Week of ', a);
      const body = document.createElement('div');
      body.innerHTML = it.body_html || '';   // rendered and sanitized by GitHub
      d.append(s, body);
      el.append(d);
    });
  } catch (e) {
    el.innerHTML = '<p>Couldn\'t load the weeklys right now (GitHub rate limit?), find them <a href="' + all + '">on GitHub</a>.</p>';
  }
})();
</script>
