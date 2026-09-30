# Agent Note: The 2026-09-30 registration round

Status: implemented

## Problem

The 2026-09-30 maintenance round revisited all three open registration pull
requests: #7 (dsh-zhipu-mcp) and #8 (dsh-550c-boot), both held by the previous
round, and #9 (dsh-widgets), opened since. All three entries pass the index gate
on their heads, so the validator again had nothing to say about the questions
that decide whether an entry ships. Two of the three carry contributions the
round could only settle with evidence the validator cannot produce: an actual
installation into an isolated DSH home, and a source-level reading of how the
plugin resolves the host services it bridges.

## Decision

**No entry was merged. Each keeps a single, named next step.**

**#8 (dsh-550c-boot): the previous round's blocker is fixed, but the entry is
still held.** The darwin app-region fix is real and shipped - the guard
`html[data-platform="darwin"] body>.dsh550c-host{-webkit-app-region:initial
!important}` is in v0.1.1 and in the current head, with the right specificity
against the official `html[data-platform=darwin] body>:not(#root)` rule. (The
previous note quoted that official selector without its platform gate; the gate
is there and the plugin matches it.) What the round found instead is that the
host element now carries `data-dsh-boot-splash`, which is **this family's**
boot-shield marker, not a host marker:

- The family stylesheet targets it directly -
  `[data-dsh-boot-splash]{position:fixed;inset:0;z-index:9999;background:var(--dsw-alias-bg-base,#1e1e20);pointer-events:none;transition:opacity 160ms ...}`
  in `packages/dsh-web-all/src/client/index.ts`. An outer-tree normal
  declaration beats the shadow root's `:host` rule, so the plugin's overlay
  loses click-to-skip (`pointer-events: none`), its own background, its z-index
  and its transition timings.
- `installBootShield()` in the same file does
  `document.querySelector('div[data-dsh-boot-splash]')`, adopts that element,
  marks it ready within one second and removes it 180 ms later. Whichever
  order the two bundles load in, the splash either dies about a second in or
  inherits the family styling above.

Two smaller items ride along: v0.1.4's `data-window-drag` on `#hud-top` sits
inside the plugin's shadow root, where neither the official drag rule nor the
shell's own `querySelectorAll('[data-window-drag]')` can see it, so the
documented "draggable during the splash" does not happen; and `dependencies`
carries `ajv`, which no source file imports. The missing reproducible tests
were raised as a request, not a gate.

**#7 (dsh-zhipu-mcp): still blocked upstream, with a second blocker recorded.**
Upstream is unchanged at `845522cf` and still declares no `dsh` field. The round
added the reason a `dsh.bundle` alone would not be enough: `index.js` resolves
`@deepseek-ai/dsh-mcp-client` by trying `DSH_APP_ROOT` and three hard-coded
Desktop install paths, then falling back to
`import('node_modules/@deepseek-ai/dsh-mcp-client/lib/index.js')` - a bare
specifier that Node rejects with `ERR_MODULE_NOT_FOUND: Cannot find package
'node_modules'`. None of the candidate paths exists on the standard install
here (`DeepSeek Harness.app` has no `Resources/app`), and the call sits
uncaught in `apply()`, so the plugin would throw on activation even if the
bundle wiring were fixed. Both were posted to the pull request, together with
the observation that the README's own first install command points at an npm
package that was never published.

**#9 (dsh-widgets): no blocker found; left open as an explicit decision item.**
The entry is compliant, the plugin is substantial rather than a wrapper (55
widget units behind nine host routes, host half 72 KB, client half 785 KB), it
declares both halves and `exports["./client"]`, its `files` whitelist excludes
`src/`, it ships no install-time script, and since 1.5.0 it publishes through
OIDC with npm attestations and SLSA provenance. An isolated install of
`dsh-widgets@1.8.2` put the name into the profile's `dsh.profile.bundles`,
which is the exact check #7 fails. What the round could not verify is the
runtime the entry promises: the live panel in the current host, the 0.1.7-rc.2
claim (only the semver arithmetic was checked against the host's own
`includePrerelease` comparison), and coexistence with `dsh-quick-ask` on the
right edge. The round reported those as unverified and asked for either runtime
evidence or an explicit maintainer go-ahead, rather than passing them silently.

## Alternatives considered

- **Merge #9 on the strength of the install and contract checks.** Rejected. The
  compatibility axis of the review owns the runtime claim, the round could not
  reproduce it, and the rule for this channel is that an unverified axis becomes
  a named decision rather than a footnote.
- **Keep blocking #8 on the item the previous round named.** Rejected as a
  mis-statement of the state: that item is fixed. The new blocker is the same
  *class* of defect - a body-level overlay colliding with a published family
  interface - so it is held on the family marker, not on a re-run of the old
  objection.
- **Treat #8's `data-dsh-boot-splash` as harmless because the family shield
  removes itself anyway.** Rejected. Removal is the harm: it deletes the
  plugin's overlay, and adoption is not guaranteed, so both load orders are
  broken in different ways.
- **Re-post #7's hold without the new loader evidence.** Rejected. The loader
  finding changes what upstream must fix; leaving it out would have produced a
  half-fix and another round trip.
- **Close #7 as stalled after five days of silence.** Rejected, as before: the
  contributor's index work is correct and the entry needs no rework from them.

## Consequences

- The catalog stays at 125 entries; nothing in this repository shipped this
  round, and the dsh-web pin for this submodule is unchanged because the round
  only added notes the market build never reads.
- Each pull request carries one next step: #8 opt out of the family marker and
  declare its own overlay properties, #7 fix the bundle declaration and the
  service lookup, #9 supply runtime evidence or an explicit go-ahead.
- A first-time contributor's CI run sits in **Action required** until someone
  with write access approves it, so the round approved #9's run before reading
  its result rather than treating the empty check list as a failure.
