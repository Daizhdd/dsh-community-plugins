# Agent Note: The 2026-10-02 registration round

Status: implemented

## Problem

The 2026-10-02 round took over the four open registration pull requests: #7
(dsh-zhipu-mcp) and #10 (dsh-screen-capture-record-desktop), both held by the
previous rounds with named steps, plus two same-day registrations, #11
(dsh-novel-writer) and #12 (dsh-nexttavern). The three-axis question is
unchanged, and two of the four turned on the host-compatibility question the
index has not had to decide before: whether an entry may be listed when the
plugin's own manifest pins a host line the current ecosystem no longer runs.

## Decision

**#11 and #10 are merged. #12 is held on host incompatibility; #7 is unchanged
and still waiting on upstream.**

**#11 (dsh-novel-writer): merged as `91752d6`.** All three axes clear on
evidence. The package declares `dsh.bundle.patch` pointing at
`cordis.patch.yml` (present in the published tarball) and pins no
`@deepseek-ai/dsh-*` peer, so the host's compatibility preflight admits it;
the 18-tool claim matches `ALL_TOOLS`, the 24 MB local model is
`BAAI/bge-small-zh-v1.5` quantised to 24,010,842 bytes, and the embedding runs
in-process over `onnxruntime-web` WASM with no subprocess and no network
download. Usefulness is a real capability the index lacks (Chinese web-novel
writing workflow, distinct from the existing read-oriented knowledge entries).
The chosen `knowledge / reading` category is legal and close enough; there is
no `writing` subcategory.

**#10 (dsh-screen-capture-record-desktop): merged as `279beb4f` after a
resolved conflict.** The three items the 2026-10-01 round named are all
delivered: the annotated `v0.5.0` tag and Release, three real-machine `.webm`
recordings in the release assets (verified reachable and playable), and a
`SECURITY.md` covering process arguments, temp-file cleanup and error paths.
The host-half source was read directly and matches the disclosed design: every
subprocess is `spawn(cmd, argvArray)` with no `shell: true` and no command
string concatenation, page-supplied numbers pass `clampInt`, the capture temp
directory is created with `mkdtemp` and removed in a `finally`, and script
paths come from `path.join` on the package directory rather than from request
input. `dsh.bundle` is declared and `dsh-screen-capture-record-desktop@0.5.0`
resolves on npm. Two non-blocking follow-ups are recorded on the pull request:
`SECURITY.md` claims `/save` returns only the file name while
`lib/index.js:494` returns absolute paths, and the `v0.5.0` tag points at a
commit that predates the published tarball. Merging #11 first made #10 conflict
because both append at the tail of `community.json`; the resolution keeps both
entries and was validated with `community-index: OK (129 entries)` plus the
9-test suite before being pushed to the contributor's branch.

**#12 (dsh-nexttavern): held - the pinned host line makes the plugin
unmountable.** The two points the description argues are not problems: a
`.json` patch file is legal (the loader accepts `.json` / `.yaml` / `.yml`,
and `__jsExpr` is the JSON form of the YAML `!!js` tag), and the index really
does not define `rank`. The blocker is that 74 `@deepseek-ai/dsh-*`
peerDependencies are pinned to exactly `0.1.7-rc.2`, while the current line is
`0.2.0-rc.2` (the repository's own README badge states `DSH >=0.2.0-rc.2`).
Evaluating the released manifest against `0.2.0-rc.2` yields INCOMPATIBLE with
74 offending peers, which is a hard stop, not a warning: `loadProfileDirectory`
throws and the bundle lands in `skippedBundles`, so the plugin is installed but
never mounted, and the profile preflight would disable the row even if the
bundle layer were bypassed. The same patch also disables and replaces six
official plugins with bundled forks marked
`formalForkSynchronized: false` against `0.1.7-rc.2`, which would displace the
official providers in a 0.2.x profile. Named next step: move to a compatible
range and verify a real start on the current line, or release a 0.2.0-rc line
and supply that version's evidence.

**#7 (dsh-zhipu-mcp): unchanged.** The upstream repository is still at its
single 2026-09-25 commit `845522cf`, with no `dsh` field, no
`cordis.patch.yml`, no tag and no npm package, and its `index.js` still
resolves `@deepseek-ai/dsh-mcp-client` through hard-coded desktop paths plus an
illegal bare specifier. Both original blockers survive; the entry itself is
compliant and conflict-free, so it can be re-reviewed as soon as upstream
changes.

## Alternatives considered

- **Listing #12 with a description note that it needs an older host.**
  Rejected. The three-axis rule asks whether the entry ships something a user
  can run; an entry that no current user can mount would send every reader to a
  plugin that does not load, and the store copy would be the only thing telling
  them so.
- **Refusing #10 because the tag and the published tarball differ.** Rejected
  as the deciding factor. The merge gate is about whether the shipped plugin
  works and is safe; the tag/artifact mismatch is a release-hygiene defect that
  was recorded as a follow-up, and the npm artifact itself did contain the
  reviewed `SECURITY.md`.
- **Resolving the `community.json` conflict by taking one side.** Rejected.
  Both sides are append-only declarations of a different entry; keeping both
  preserves each intent exactly.

## Consequences

- Two entries are adopted into the workshop and two stay open with a named next
  step; the index now carries 129 entries.
- The round records a durable rule the index had not written down: an entry
  whose plugin pins `@deepseek-ai/dsh-*` peers to a host line outside the
  supported range describes a plugin the current ecosystem will refuse to
  mount, so host-line compatibility is a listing gate, not a copy detail.
- `community.json`'s tail is a shared append point for concurrent
  registrations; future rounds should expect the same conflict and resolve it by
  keeping both entries.
