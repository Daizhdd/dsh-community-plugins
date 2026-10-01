# Agent Note: The 2026-10-01 registration round

Status: implemented

## Problem

The 2026-10-01 maintenance round took over the four open registration pull
requests: #7 (dsh-zhipu-mcp) and #8 (dsh-550c-boot) held by the previous
rounds, #9 (dsh-widgets), and #10 (dsh-screen-capture-record-desktop), a
same-day repository. The question each entry faces is unchanged - the index
gate only proves the JSON shape, while usefulness, upstream stability, and
compatibility decide whether an entry ships - and this round had to settle two
of them plus one first-of-its-kind merge conflict.

## Decision

**#8 and #9 are merged and adopted into the workshop; #7 and #10 stay open with
a single named next step each.**

**#8 (dsh-550c-boot): merged.** The author answered the 2026-09-30 review point
by point on 2026-10-01. The blocking item - borrowing the dsh-web-all family
marker `data-dsh-boot-splash` - is fixed in upstream v0.1.1 through v0.3.3: the
host element now carries its own `data-dsh-550c-boot` with an explicit
`aria-hidden`, and the built `lib/client.js` was grepped directly (not the
stale code-search index, which still lists the old marker in docs and
CHANGELOG). The entry description matches v0.3.3 behavior and the `npm` field
is filled (`dsh-550c-boot@0.3.3` on npm). CI was green on the final head after
the first-time-contributor runs were approved.

**#9 (dsh-widgets): merged after a resolved tail-append conflict.** The
three-axis assessment cleared it on its own evidence: a versioned release line
with changelogs (v1.7.0 - v1.8.3), published types, verification on both the
0.1.x and 0.2.x SDK lines, and an npm field that resolves. #8 and #9 both
append at the tail of `community.json`, so merging #8 first made #9 DIRTY; the
resolution keeps both entries in one round (550c-boot from main, widgets from
the branch) and was validated with `community-index: OK (127 entries)` plus the
9-test suite before being pushed to the author's branch. Both intents are
append-only declarations, so keeping both provably preserves each side; no
logic was traded away.

**#7 (dsh-zhipu-mcp): held - stability evidence is insufficient.** The upstream
repository has had no commit, release, or npm package since its creation day.
Usefulness is affirmed; compatibility has a real concern (the plugin binds the
host's `dsh-mcp-client` at strict version parity, so host upgrades can break it
silently) that the entry copy should surface. Named next step: a tagged
version and at least a week of upstream maintenance evidence, plus an explicit
failure-mode note.

**#10 (dsh-screen-capture-record-desktop): held - stability and the host-side
execution path are unverified.** The repository was created the day the PR
opened, with no release. It is the only entry in this round whose plugin
executes local processes (Python, ffmpeg) in the host half; reading that
implementation is a prerequisite, not a formality. Named next step: a version
tag, a full-screen recording produced on a real machine, and a source review of
process arguments, temp-file cleanup, and error paths.

**Adoption.** Both merged entries are index-only content, so the round pinned
`satellites/dsh-community-plugins` at the merge commit (2fc8fc19) and rebuilt
`market/dist` in dsh-web (127 plugins); the dsh-web shelf commit c42e3d24 went
green on deploy-market.

## Alternatives considered

- Merging #7 with a caveat: rejected - the standard for shipping an entry to
  every Workshop user is evidence of maintenance, and a day-one repository
  cannot show it; the entry keeps its review and stays open.
- Merging #9 with `--auto` and letting GitHub resolve after #8: rejected - the
  tail-append conflict is not auto-resolvable, and leaving the conflict to the
  author would have stalled two cleared entries on bookkeeping.
- Marking #10 unverified but mergeable because the entry copy discloses its
  limits: rejected - disclosure addresses usefulness, not the unreviewed
  host-side process execution.

## Consequences

The index carries 127 entries and the workshop serves both new plugins. #7 and
#10 hold reviews that name exactly what unblocks them, so the next round can
re-evaluate on evidence rather than re-litigate. The tail-append conflict is a
known shape now: every same-day pair of registrations will produce it, and the
resolution recipe (keep both, run the gate, push to the branch) is recorded
here.
