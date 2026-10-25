# Burler plan round 2 — fixer report

Every finding's premise held against the code.
F1, F3 and F4 were fixed as proposed, with refinements from the self-check; F2 was fixed differently in its ownership gate.
`lyx loom validate-plan` reports `ok` after every change.

## Changes

- **F1 (BLOCKING), fixed as proposed, refined.**
  Premise checked: `judgeOrigin` in `internal/battencli/cancel.go:396-405` returns `cancelOwnedElsewhere` for a present worktree with a reachable origin, and `classifyEntry` puts `owned_elsewhere` ahead of `stopped`.
  Card 15 creates `UpstreamRunUnfinished`, which reads the pair's own run status at the fetched upstream tip's tree: `TreeRunFileRels` of the slug, with the legacy `self` fallback, joined under `treeAnchor` of the tip, since `TreeRunFileRels` returns anchor-relative paths.
  The signature takes `(pair *lyxcwd.Location, names []string)` instead of the proposed `(weftPath string)`, because the reader needs the slug for the run id and the wired names plus a fallback for `treeAnchor`.
  Card 17's unfinished-run refusal now defines the run status (the worktree's own status, plus for a beam the prime's batten status of the slug, each unfinished when its `state` is other than `done`).
  For an owned-elsewhere worktree it judges the worktree's own status at the upstream tip, and names `lyx batten cancel <slug>` on the owner machine, then `lyx fabric pull` to fetch the new tip, then the `--set`.
  The driver-lock and batten-run refusals are unchanged.
  New rows: in card 17's `TestChangeAnchor` (stale-unfinished checkout with a finished upstream is let through, an unfinished upstream refuses, a prime batten status refuses) and in card 15's `TestAnchorChange`.
  The overview's way-forward line follows.
- **F2 (BLOCKING), fixed differently.**
  Card 14 creates `resetWeftToIntegrated`, a `destroy.go` executor with a leading `rec *Mutations`.
  `integrateWeft` moves the branch through it and never calls `reset --keep` itself.
  `TestIntegrateWeft` gains a row: a dirty tracked path that the result rewrites is refused, and the branch is unchanged.
  The proposed ownership gate, "a registered linked worktree", was not adopted.
  That predicate excludes the repository's main checkout (`ownedRegisteredLinkedWorktree`'s doc, `destroy.go:327-331`).
  Card 17's `TestRunCLI_Anchor` integrates the prime's weft, and the existing weft replay gates that weft with `ownedWarpCheckout(worktree)` (`replay.go:437`), which admits any worktree, the main checkout included.
  The executor therefore uses the weft replay's own request (`replayRequest` over `ownedWarpCheckout(weftPath)`).
  That keeps containment, dirtiness on the tracked paths `onto` rewrites, and no force.
  The stale `cherryPickOnto` was dropped from card 14's Uses.
- **F3 (MEDIUM), fixed as proposed.**
  Card 13 creates `RemoveMoveRecord` in `internal/fabricengine/moverecord.go`, a tolerant `os.Remove` following `ClearPushFailure`.
  It edits `cmd/lyx#destructiveGuardAllowlist` with that file's reason.
  The three removal sites in cards 15 (`Apply`'s failure path, `Finish`) and 18 (Add) call it.
  Card 13's ImpactSummary names the allowlist entry.
- **F4 (MEDIUM), fixed as proposed, then reworked after the self-check.**
  Card 15: `Apply` returns `landed` false with the record removed only once the pair is wired at `A` again.
  A failed restore, `.lyx` rename or rewire keeps the record and names the re-run.
  `Close` removes a record this `BeginAnchorChange` wrote fresh when `Apply` never ran, and keeps a record a re-run found standing.
  Card 17 decides the restart by the record after `Close`:
  - With the record gone, it restarts the ended dobbies. That covers the pre-`Apply` errors of a fresh run, matching D7 step 4.
  - With the record standing, it starts no dobby and names the re-run. That covers a landed or failed-restore `Apply` error, a sink-reopen error, a `Finish` error before its removal, and any error of a re-run over a standing record.
  - A `Finish` error after the removal is reported like a failed dobby start.

  Rows were added to both cards' tests, and the overview line follows.

## Self-check

- fix-claims, LOW: which run status the owned-elsewhere path reads, given the prime's batten status of the slug. Fixed: card 17 now names both statuses and reads the batten status from the prime's unstopped weft.
- fix-consistency, MEDIUM: the pre-`Apply` errors leaving the record contradicted D7 step 4 (`decision-record.md:147`). Fixed with my own change: `Close` removes a fresh record when `Apply` never ran, and card 17 restarts the ended dobbies for those errors. Merged with fix-proposal's first finding.
- fix-consistency, MEDIUM: "unfinished" was left undefined, and so was which status file. Fixed: card 17 defines it (`state` other than `done`), and card 15's reader applies the same rule.
- fix-consistency, LOW: the restore-failure row leaves both `.lyx` directories, so the named re-run refuses. Fixed: the `Apply` sentence now says the re-run recovers as the not-landed re-run does, refusal over both `.lyx` included, and the row asserts that refusal.
- fix-proposal, MEDIUM: errors before `Apply` kept the record and stopped weft writers for nothing. Fixed: merged with fix-consistency's first finding, same change.
- fix-proposal, MEDIUM: F1's way forward reads a fetched tip nothing refreshes. Fixed: the refusal names `lyx fabric pull`, whose weft arm fetches before it stops (card 9), ahead of the `--set`.
- fix-affected-set, MEDIUM: no end state for a sink-reopen error after `Apply`, or for a `Finish` error. Fixed: both are covered by card 17's record-based rule, and a `Finish` error after removal is reported like a failed dobby start, with rows added.
- fix-affected-set, LOW: card 18's Add stated no end state between the record write and removal. Fixed: the error leaves the record and names `lyx beam create <slug>`, which recovers it through `EnsureAnchor`.
- fix-affected-set, outside lens, LOW: card 14's stale `cherryPickOnto` in Uses. Fixed: removed.

## Disputed

None.

## Deferred

None.
