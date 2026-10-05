# Agent Note: The 2026-10-05 registration round

Status: implemented

## Problem

The 2026-10-05 round took the four open registration pull requests in this
repository: #15 (ten right-sidebar plugins by lemonhall, a first-time
contributor), #14 (dsh-whale-musume), #12 (dsh-nexttavern) and #7
(dsh-zhipu-mcp). #15 is a batch; the other three had a hold from an earlier
round, and each hold was a claim about the plugin's runtime behaviour rather
than about the index entry. Since this repository publishes an entry to every
user who browses the Workshop, the three-axis judgement has to rest on the
contributor's actual package, not the pull-request text.

## Decision

**No entry merged. #15 was reviewed and held on three plugins; #14, #12 and #7
were re-verified at their current upstream state and remain held on unchanged
blockers.**

- **#15 (ten plugins).** Every entry is format-correct and conflict-free
  against the 131 entries on main, and all ten declare a bundle patch and a
  client half with no pinned peers, so the entry layer is not the problem. The
  round cloned all ten repositories and ran their tests. Three carry a hard
  blocker:
  - `dsh-rss-dock` claims automatic Chinese translation of English titles and
    summaries in its entry and its pull-request description, "cached by source
    hash". No translation call or cache exists anywhere in the repository
    (`lib/index.js`, `lib/client.js`, `lib/state.js`); the only cache is
    `feedCache`, keyed by feed URL. It also hardcodes `curl: 'curl.exe'`
    (`lib/index.js:19`), so `execFile` fails with `spawn curl.exe ENOENT` on
    macOS and Linux.
  - `dsh-radio-dock` hardcodes `curl.exe` (`lib/index.js:28`) for both the
    catalog and the stream proxy - non-Windows hosts cannot use it - and its
    `/dsh-radio/stream` route forwards any `http(s)` URL from the `u` query
    parameter through `spawn` with `access-control-allow-origin: *`, which
    makes the host an unauthenticated open proxy for loopback and intranet
    targets.
  - `dsh-finance-dock` omits `"npm": "dsh-finance-dock"` from its entry even
    though 0.2.0 is published, so the Workshop degrades to a git-repository
    install; it also hardcodes `curl.exe` (`lib/index.js:37`).
  The other seven - `dsh-calendar-dock`, `dsh-ledger-dock`,
  `dsh-calorie-dock`, `dsh-todo-dock`, `dsh-pomodoro-dock`, `dsh-irc-dock`
  and `dsh-qqmail-dock` - pass: no external dependencies, state under
  `$DSH_HOME/<id>/` with an atomic rename, a `sec-fetch-site` guard on the
  host route, and their offline tests pass (11, 21, 23, 5, 14, 14 and 15
  assertions).
- **#14 (dsh-whale-musume).** The upstream repository now has tests (143/143
  pass), a bundle contract and a per-release compatibility table, so it is not
  refused. Its entry is still held on its description: `readPref` defaults
  every unset key to on (`assets/dsh-whale-moe.js:58-60`), so idle chat is
  enabled by default and reads the last conversation node's text through
  `latestTaskTopic()` (`:2660-2666`, `:3061`), and the low-balance hint
  reads the host's shared `dsh.balance.low` marker (`:3183`) regardless of the
  balance toggle. "Every toggle that reads conversation content or account
  balance is off by default" does not match the source.
- **#12 (dsh-nexttavern).** The blocker is unchanged: the published 0.2.9
  package pins 74 `@deepseek-ai/dsh-*` peers to exactly `0.1.7-rc.2`, and
  current main pins 79. On the 0.2.0-rc.2 host that is judged incompatible and
  the bundle lands in `skippedBundles`, so the entry would send every user to
  a plugin that never mounts.
- **#7 (dsh-zhipu-mcp).** The blocker is unchanged: upstream main is still
  `845522cf` with no `dsh` field, no `dsh.bundle` and no
  `cordis.patch.yml`, and `index.js` still hardcodes desktop-app paths before
  falling back to the illegal bare specifier
  `node_modules/@deepseek-ai/dsh-mcp-client/...`. Without `dsh.bundle` the
  package installs as a plain dependency and never mounts.

## Alternatives considered

- **Merging the seven sound entries of #15 and leaving the three out.** Rejected
  for this round: the pull request is one `community.json` append of ten, it is
  already conflicting with main, and splitting it is the contributor's edit to
  make. The comment names the split explicitly so the next round is a review of
  a seven-entry pull request.
- **Reading the pull-request descriptions as evidence for the three-axis
  judgement.** Rejected. The rss-dock description is the counter-example that
  proves the point: it describes a feature the code does not contain. Each
  claim was checked against the repository at its current tip.
- **Treating #14 as refused on stability because it is not on npm.** Rejected.
  The index contract makes `npm` optional, 23 existing entries omit it, and the
  plugin is a repository install. The hold is on the description, which is a
  user-visible claim.
- **Closing #7 or #12 as abandoned.** Rejected. Both are held on a concrete
  upstream change with the exact edit named; they stay open for the
  contributor.

## Consequences

- Four pull requests stay open, each with the one concrete change it needs: a
  split plus three upstream fixes for #15, an honest description for #14, a
  compatible peer range for #12, and `dsh.bundle` plus a host-relative module
  resolution for #7.
- The round records a reusable check for this repository: a registration entry's
  description is a user-facing claim, so the three-axis pass has to open the
  package and look for the feature, not only confirm that the bundle and the
  metadata parse.
- The index itself was not changed by this round.
