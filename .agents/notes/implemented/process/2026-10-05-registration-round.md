# Agent Note: The 2026-10-05 registration round

Status: implemented

## Problem

The 2026-10-05 round took the open registration pull requests in this
repository. On the first pass: #15 (ten right-sidebar plugins by lemonhall, a
first-time contributor), #14 (dsh-whale-musume), #12 (dsh-nexttavern) and #7
(dsh-zhipu-mcp). #15 is a batch; the other three had a hold from an earlier
round, and each hold was a claim about the plugin's runtime behaviour rather
than about the index entry. Since this repository publishes an entry to every
user who browses the Workshop, the three-axis judgement has to rest on the
contributor's actual package, not the pull-request text. A second pass then took
the reworked #15 and a new submission, #17 (dsh-camofox-browser).

## Decision

**No entry merged in the round. #15 was held on three of its ten plugins, then
reworked into a seven-entry pull request and held again on one description; #17
was held on its shipped configuration; #14, #12 and #7 remain held on upstream
changes.**

### First pass

- **#15 (ten plugins).** Every entry was format-correct and conflict-free
  against the 131 entries on main, and all ten declared a bundle patch and a
  client half with no pinned peers, so the entry layer was not the problem. The
  round cloned all ten repositories and ran their tests. Three carried a hard
  blocker:
  - `dsh-rss-dock` claimed automatic Chinese translation of English titles and
    summaries in its entry and its pull-request description, "cached by source
    hash". No translation call or cache existed anywhere in the repository
    (`lib/index.js`, `lib/client.js`, `lib/state.js`); the only cache was
    `feedCache`, keyed by feed URL. It also hardcoded `curl: 'curl.exe'`
    (`lib/index.js:19`), so `execFile` failed with `spawn curl.exe ENOENT` on
    macOS and Linux.
  - `dsh-radio-dock` hardcoded `curl.exe` (`lib/index.js:28`) for both the
    catalog and the stream proxy - non-Windows hosts could not use it - and its
    `/dsh-radio/stream` route forwarded any `http(s)` URL from the `u` query
    parameter through `spawn` with `access-control-allow-origin: *`, which
    made the host an unauthenticated open proxy for loopback and intranet
    targets.
  - `dsh-finance-dock` omitted `"npm": "dsh-finance-dock"` from its entry even
    though 0.2.0 was published, so the Workshop would degrade to a
    git-repository install; it also hardcoded `curl.exe` (`lib/index.js:37`).
  The other seven - `dsh-calendar-dock`, `dsh-ledger-dock`,
  `dsh-calorie-dock`, `dsh-todo-dock`, `dsh-pomodoro-dock`, `dsh-irc-dock`
  and `dsh-qqmail-dock` - passed: no external dependencies, state under
  `$DSH_HOME/<id>/` with an atomic rename, a `sec-fetch-site` guard on the
  host route, and their offline tests passed (11, 21, 23, 5, 14, 14 and 15
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
- **#12 (dsh-nexttavern).** The published 0.2.9 package pins 74
  `@deepseek-ai/dsh-*` peers to exactly `0.1.7-rc.2`, and current main pins
  79. On the 0.2.0-rc.2 host that is judged incompatible and the bundle lands in
  `skippedBundles`, so the entry would send every user to a plugin that never
  mounts.
- **#7 (dsh-zhipu-mcp).** Upstream main is still `845522cf` with no `dsh`
  field, no `dsh.bundle` and no `cordis.patch.yml`, and `index.js` still
  hardcodes desktop-app paths before falling back to the illegal bare specifier
  `node_modules/@deepseek-ai/dsh-mcp-client/...`. Without `dsh.bundle` the
  package installs as a plain dependency and never mounts.

### Second pass

- **#15 was split as asked.** Head `a47f2f5a` carries only the seven sound
  plugins, rebased onto the current main (`e7511fe`), as a pure 82-line
  addition; the held three are acknowledged and left for their own upstream
  fixes. Re-verified from scratch: `community-index: OK (138 entries)`, the
  validator's own tests 9/9, no duplicate ids, every category/subcategory legal,
  and each of the seven packages re-cloned and re-tested at its current tip
  (exit 0 across the board). The entry layer is now correct.
  The rework is held on one remaining item: the `dsh-irc-dock` entry says
  "connects to Libera.Chat by default", but the shipped default is EFnet. The
  package's `cordis.patch.yml` sets `config.server: 'irc.efnet.org:6667'` (and
  its own comment calls EFnet the default), while `apply()` computes
  `{ ...DEFAULTS, ...cfg }` and connects on `persisted.server || opts.server`;
  a fresh install has no persisted value, so EFnet wins and the library's
  `DEFAULTS.server = 'irc.libera.chat'` is overridden. The two are different
  networks, so a user who trusts the entry lands somewhere other than expected.
- **#17 (dsh-camofox-browser) was held on its shipped configuration.** The
  three-axis read is practical (anti-detection browsing is a real need, and
  `tools / browser` holds only `dsh-pilot`), and the upstream is healthy
  (`jo-inc/camofox-browser`, MIT, 11.4k stars, released v1.18.1 the same day),
  with a smoke test that passes 21/21 locally. The blocker is compatibility: the
  published `cordis.patch.yml` and the library's `DEFAULTS` both use the
  author's own absolute Windows paths - `serverDir: 'E:\\development\\camofox-browser'`,
  `tempDir: 'E:\\caches\\tmp'`, `screenshotDir: 'E:\\development\\dsh-camofox-browser\\shots'` -
  and the repository ships only `install.ps1`. Any other user, including on
  Windows, reaches the "cannot find the camofox-browser service" throw; on
  macOS/Linux the path cannot exist at all (confirmed locally:
  `existsSync(serverDir/server.js)` is false and `ping()` returns false).

## Alternatives considered

- **Merging the seven sound entries of #15 on the first pass.** Rejected then:
  the pull request was one `community.json` append of ten and already
  conflicting with main, so splitting it was the contributor's edit to make. The
  comment named the split so the next pass would review a seven-entry pull
  request - which is what happened, and it is now one description line from
  mergeable.
- **Reading the pull-request descriptions as evidence for the three-axis
  judgement.** Rejected. The rss-dock description is the counter-example that
  proves the point: it described a feature the code did not contain. Each claim
  was checked against the repository at its current tip - which is also how the
  irc-dock default mismatch surfaced, in a batch that had already passed on
  every other axis.
- **Treating #14 as refused on stability because it is not on npm.** Rejected.
  The index contract makes `npm` optional, 23 existing entries omit it, and the
  plugin is a repository install. The hold is on the description, which is a
  user-visible claim.
- **Closing #7 or #12 as abandoned.** Rejected. Both are held on a concrete
  upstream change with the exact edit named; they stay open for the
  contributor.
- **Accepting #17 because the plugin builds and its smoke test passes.**
  Rejected. A build and a self-test do not show that the shipped defaults work
  anywhere but the author's machine, and a registration entry promises every
  Workshop reader that the plugin does something.

## Consequences

- Five pull requests stay open, each with the concrete change it needs: one
  description line for #15, an honest description for #14, a compatible peer
  range for #12, `dsh.bundle` plus host-relative module resolution for #7, and
  platform-neutral defaults (or an explicit Windows-only statement) for #17.
- The round records a reusable check for this repository: a registration entry's
  description is a user-facing claim, so the three-axis pass has to open the
  package and look for the feature and its defaults, not only confirm that the
  bundle and the metadata parse.
- The index itself was not changed by the round.
