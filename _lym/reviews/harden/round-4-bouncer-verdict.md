---
verdict: CONVERGED
rationale: "Round 4 carries no frontmatter finding, no BLOCKING or MEDIUM finding on a key open in an earlier round, and a green live gate; O1 (MEDIUM, design) is a key first raised in round 4, so it rides as an open risk with O2 to O5 and the undriven paths."
---

## Ruling

The facts show zero findings in rounds 3 and 4, the live report green, no strays and no fix commits.
The decision rule gives `CONVERGED` when the latest round carries no BLOCKING finding and no gating finding at MEDIUM or worse on a key open in an earlier round.
The facts' list of keys open in an earlier round holds only keys from rounds 1 and 2, all resolved in the round 3 ledger.
No round 4 finding maps to any of them.

## Open risks

The review records O1 to O5 as observations below the frontmatter, because the round was incremental.
Each sits on a key first raised in round 4, so none blocks convergence.
The ledger carries each as `open`.
`ly:orch` (creel #2422) filed them on the board as note `harden-fabric-undriven-cases` for a follow-up task and asked that the verdict name them.

- O1, MEDIUM, design: a `lym fabric clone` killed after its records store exists and before `_bolt` does leaves a hub whose printed way forward fails.
  `lym fabric reconcile` fails with a raw `gitrepo:` text, and a plain re-clone answers `hub already exists` with no way forward.
  Only `--reset`, which no output names, rebuilds it.
  Site: `internal/fabriccli/fabric.go:673`, `internal/fabricengine/clone.go:225` and `:264`.
  I read O1 under the rubric as a MEDIUM that would keep a full-scope round from converging; the incremental scope and the orch's settlement carry it forward instead.
- O2, LOW, scope: `lym fabric merge` stages a merge with the records store's fetch refspec unset and names no layout gap (`internal/fabricengine/mergeguards.go:57`).
- O3, LOW, design: `pushLocalArchiveTags` pushes tags `DropLocal` kept local when a later `--remote` teardown runs (`internal/fabricengine/archive.go:294`).
- O4, LOW, scope: a plain `refs/tags/archive/` ref on a one-repository origin is named by no verb (`internal/fabricengine/cleanupremotewarp.go:302`).
- O5, NIT, consistency: a killed discard's way forward names no command an operator can type (`internal/fabricengine/remoteteardown.go:187`).

## Focus departures

Round 4 lists four departures.
Ratified:

- Focus 4, tags placed by hand at the refs `pushArchiveTag` spells: staging the real path risks a real session the cost declaration bans, and the read side was driven in two hubs.
- Focus 8, hand-over and code-tip reset paths undriven: the `hubforge` seam is test-support code no sandbox verb reaches.
- Focus 2's repro, a hand-made tag standing in for `DropLocal`'s: a plain removal pushed its tag, and the sandbox cannot stage a takeover.
- Recording O1 to O5 in prose below the frontmatter: the format's incremental rule admits no other place.

I rejected none.
No finding was relabelled.

## Coverage

Not driven: the owner-archive push path, the hand-over and code-tip reset paths of `handover.go` and `destroy.go`, a lease that loses, and a kill inside `fabric checkout`.
`cleanup --apply --remote`'s warp sweep never ran, since a sandbox origin skips the GitHub pull-request listing.
The method showed which ref a push names, not that nothing else wrote origin.
The concurrency race found no contention, so it proved the serialised path only.
The focus 7 table rests on caller tracing; only the `lym fabric` gap was driven.
The review's own `verdict: APPROVED` is no signal, and the ruling rests on the facts and the findings.
