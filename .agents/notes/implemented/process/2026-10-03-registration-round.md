# Agent Note: The 2026-10-03 registration round

Status: implemented

## Problem

The 2026-10-03 round took over the four open registration pull requests in this
repository: #7 (dsh-zhipu-mcp), #12 (dsh-nexttavern) and #14
(dsh-whale-musume), all held by earlier rounds or new, plus #13
(dsh-as-aistudio). The three-axis question from the 2026-10-02 round is
unchanged. Two of the four turned on things the previous rounds had not had to
decide: a store description whose stated default boundary contradicts the
plugin's own source, and whether an upstream that has published a newer release
line actually cleared a host-line hold.

## Decision

**#13 is merged. #14 is held on an inaccurate default-behaviour description;
#12 and #7 are unchanged, still waiting on upstream.**

**#13 (dsh-as-aistudio): merged as `7f76bda2`.** All three axes clear on
evidence. The entry is a single append to `community.json` with valid
`tools` / `context` enums; `community-index.cjs --check` reports OK (130
entries) and the 9-test script suite passes. Upstream is live (v0.2.4, commits
through the merge day) and every dependency resolves on npm: dsh-edit-turn
0.2.16, dsh-rerun-turn 0.1.24, dsh-delete-turn 0.1.7, dsh-markdown-bubble
0.1.4. The aggregate entry is not a duplicate: no existing entry carries it or
any of its four components. Its declared bundle (`dsh.bundle.patch` ->
`cordis.patch.yml`, present) and `dsh.client` web half are real, and the
component fan-out is covered by a 16-combination mount matrix with per-component
fault isolation. Usefulness is a real capability the index lacks — turn-level
prompt editing, in-place re-run and turn removal, which the official `/compact`
does not offer.

**#14 (dsh-whale-musume): held - the description contradicts the source.** The
entry is schema-valid (single append, `ui` / `panel` legal,
`community-index: OK (130 entries)`) and upstream's `npm test` reproduces
143/143 with no skips. What fails is the copy the store shows every user, which
this repository reviews as user-visible: the entry states that "every toggle
that reads conversation content or account balance is off unless you turn it
on", but the default-on chat path calls `latestTaskTopic()`, which reads the
last conversation node's text
(`assets/dsh-whale-moe.js:2660-2666`, reached from the default-enabled
`idleChatTick` at `:3061-3098`); the `keywords` toggle governs a different
path. The balance toggle likewise only stops the amount poll, while the shared
low-balance marker is still read at `:3183`. The client half also registers
DOM, global listeners, an interval and a MutationObserver that are not attached
to the host's `ctx.effect` disposal, and its boot flag is not reset on failure.
Named next step: align the description with the real default boundary (or add a
single explicit opt-in gate upstream), and approve the CI run - which sits in
**action_required with zero jobs**, so the index gate has not executed at all on
this head.

**#12 (dsh-nexttavern): unchanged - the exact host pins survive the new
releases.** Upstream has moved: its `main` is now `2cf1db44` with commits on
the review day, and it has published `v0.2.9-rc.2` (2026-10-03). The blocker
is unaffected, because the released line still pins the old host: npm
`dsh-nexttavern@0.2.9` carries the same exact `@deepseek-ai/dsh-*` peers at
`0.1.7-rc.2` (74 of them), and upstream `main`'s `0.3.0-rc.1` pins 79, also
exact — the release that npm serves and the development line both remain
unmountable on the `0.2.0-rc.2` ecosystem. The follow-up comment therefore
lands on the new head rather than the commit the 2026-10-02 round reviewed. The
entry is still conflict-free in itself and the branch is conflicted only on the
`community.json` tail, which moved when #13 merged.

**#7 (dsh-zhipu-mcp): unchanged.** Upstream is still at its single 2026-09-25
commit `845522cf`, with no `dsh` field, no cordis patch and no npm package;
both original blockers survive and the entry itself remains compliant.

## Alternatives considered

- **Listing #14 on the strength of its 143-passing test suite and letting the
  store copy stand.** Rejected. The description is the only thing a user reads
  before installing, and here it makes a privacy claim the code contradicts;
  accepting it would set the precedent that default-behaviour copy is not
  enforced against source. The plugin is genuinely capable and the hold names
  two concrete paths, so this is a copy-or-gate fix, not a rejection.
- **Treating #12's new upstream activity as clearing the host-line hold.**
  Rejected. "The repository is active" is not the same as "the released
  artifact mounts": the npm `latest` still pins `0.1.7-rc.2` exactly, and the
  fresh `0.2.9-rc.2` release is an installer/distribution build, not a
  re-targeted host line. The 2026-10-02 rule stands: a plugin that pins the
  supported host out is not listable.
- **Approving #14's CI run and merging on the resulting green.** Rejected.
  A green index gate proves the JSON is well-formed; it says nothing about the
  description's accuracy, which is the actual blocker.
- **Holding #13 for a local install-and-run.** Rejected as disproportionate.
  The index entry only links and describes; the components' interop contract is
  already covered by the upstream combination matrix, and no repository rule
  requires a store-only entry to be installed to merge.

## Consequences

- One entry is adopted (the index now carries 130) and three stay open with a
  named next step each.
- The round records a second durable listing gate alongside the 2026-10-02
  host-line rule: an entry's description of defaults, permissions and platform
  support is checked against the upstream source before merge, and a privacy or
  permission claim that the code contradicts blocks the entry the same way a
  broken manifest does.
- `community.json`'s tail remains a shared append point: merging #13
  conflicted #12, exactly as the 2026-10-02 round predicted. A round that merges
  more than one registration should expect to resolve the tail by keeping both
  entries, never by taking one side.
