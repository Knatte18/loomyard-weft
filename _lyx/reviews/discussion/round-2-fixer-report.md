# Round 2 fixer-report — discussion

## Changes

- F1 MEDIUM, fixed as proposed.
  D3 no longer claims a boot marks an existing session: a boot marks only the session it spawns, and a prime session already up when the new binary arrived is marked at its next attach pre-flight.
  It adds the bound that until such an attach, the session draws in the second loop.
  The acceptance criterion now reads "is marked after an attach to it, and a prime session a fresh boot spawns is marked by that boot".
  The bound names the attach verb as `lyx reed attach`, the only production caller of `AttachArgv` (`internal/reedcli/attach.go:105`).
- F2 LOW, fixed as proposed.
  D2 now says "The four probes, the three stopped-empty conditions and the comparison of the picked commit's tree with its first parent's, are CLI calls through `runChecked`".
  The self-check then extended this sentence (see below).
- `support-log.md` gains a Round 2 section that records both findings.
  F1's entry says it replaces the earlier "the option is set on boot too" for an existing prime session.

## Self-check

- fix-consistency, LOW: support-log's earlier "the option is set on boot too" contradicted the new D3. Fixed: the Round 2 F1 entry now says it replaces that claim.
- fix-claims, MEDIUM: the D3 bound wrongly implied that every attach marks the prime, when `AttachArgv` degrades to the bare argv before reaching `pinGeometryOptionsLocked` (`attach.go:31-35`, `:97`). Fixed: the bound now covers only an attach whose pre-flight reaches the geometry pins, and it states that a degraded attach leaves the session unmarked.
- fix-proposal, LOW: D2's "on the gitrepo client boundary's list of CLI reads" did not explain why the tree comparison, a local object read, is not done with go-git. Fixed with my own change instead of the fork's go-git option: the probes run inside `CherryPick`, which `internal/gitrepo/doc.go:25` already lists as CLI-bound and `cmd/lyx/gitrepoboundary_test.go:72` admits. So they add no exported method, and the sentence now says this.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.
