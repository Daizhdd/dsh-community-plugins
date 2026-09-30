# Agent Note: The 2026-09-29 registration round

Status: implemented

## Problem

The 2026-09-29 maintenance round reviewed the two open registration pull
requests in this repository: #7 (dsh-zhipu-mcp) and #8 (dsh-550c-boot). Both
entries are structurally correct and both passed the index gate on their heads,
so the structural validator had nothing to say about them. The questions that
decide whether an entry may ship - whether the plugin actually loads once
installed, and whether it coexists with the host shell - are outside what the
validator can see.

## Decision

**Neither entry was merged; both were held with their specific blocker named.**

**#7 (dsh-zhipu-mcp) is blocked upstream, unchanged from the previous round.**
`package.json` in `youbuwei/dsh-zhipu-mcp` still carries no `dsh` field at all,
so there is no `dsh.bundle`: `dsh plugin add` installs the package as a plain
dependency, the reconcile step skips it, it never enters the profile's
`bundles`, and a restart does not mount it. The entry is compliant and the
plugin's internals are sound; the store would still be pointing every user at a
plugin that installs without taking effect. Upstream has not been pushed since
2026-09-25, so the round recorded the state and left the pull request open. The
advisory note about `APP_ROOTS` hardcoding a Desktop-only install path was
repeated as advice, not as a gate.

**#8 (dsh-550c-boot) is blocked on one host-shell regression.** The plugin is
otherwise the strongest kind of community entry: no install-time lifecycle
scripts, a `files` whitelist that excludes `src/`, a committed `lib/client.js`
that rebuilds byte-identically from committed `src/`, CI that enforces that, no
network surface in either half, and reliable exit paths (click, Esc, rejection,
throw and a 30 s/12 s watchdog all funnel into one teardown). The blocker is
that it mounts a viewport-sized `body > .dsh550c-host` carrying none of the
three markers the family counter-guard keys on, while the official sheet
declares `html[data-platform=darwin] body>:not(#root){-webkit-app-region:no-drag}`.
That subtracts the whole window from the macOS draggable region while the
splash plays - the exact regression this ecosystem recorded on 2026-09-25. The
fix is one attribute, and it belongs upstream.

The engine floor mismatch (`dsh.engines.dsh: >=0.2.0-rc.1` against the
satellites' `>=0.1.7-rc.1` and the 0.1.7-rc.2 host in use) was reported as
advisory rather than blocking: the dsh-web aggregate on its dev line already
declares `>=0.2.0-rc.1`, so the plugin is tracking a real target, and only the
in-app update path is refused on the older host.

## Alternatives considered

- **Merge #8 and ask for the app-region fix in a follow-up.** Rejected. The
  regression is in the family's own recorded failure mode, it is one attribute,
  and the entry is what sends the install command to every user. An entry does
  not get to ship a known host-shell regression on a promise.
- **Treat the `-webkit-app-region` collision as the plugin's private business
  because it only affects macOS.** Rejected. The counter-guard exists precisely
  because body-level overlays are a shared surface; the marker list is the
  interface the family publishes for them.
- **Block #8 on the engine floor as well.** Rejected. The floor tracks the
  dsh-web dev line, the mismatch affects only updates on the older host, and
  blocking on it would have obscured the one item that must actually change.
- **Close #7 as stalled.** Rejected. The blocker is upstream's to clear, the
  contributor's index work is correct, and the entry needs no rework from them.

## Consequences

- Neither entry enters the catalog this round; the store continues to serve the
  current 125 entries.
- #8 has a single, actionable ask: opt the host element out of the app-region
  computation. #7 waits on an upstream `dsh.bundle` declaration.
- The two pull requests stay open with their blockers recorded, so the next
  round reads the state rather than re-deriving it.
- Both states moved on 2026-09-30 (see that round's note): #8's ask was met and
  the hold moved to the family boot-splash marker, and #7 gained a second
  blocker in how the plugin resolves the host's mcp client.
