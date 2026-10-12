# Fixer report — plan, round 1

Every finding's premise held against the cards and code; all seven are fixed.
`lyx loom validate-plan` passes after the edits with informational findings only.

## Changes

- **F1** (fixed as proposed): card 16 lists `internal/battenshed#battenshedAllowedImports` under **Edit:** and its Intent says it gains `internal/pauserecord`; card 19 does the same for `internal/shedverbs#shedverbsAllowedImports`; card 21 for `internal/orchengine#orchengineAllowedImports`, gaining `internal/pauserecord` and `internal/agentname`.
  Card 20's shedverbs imports are covered by card 19's addition, which lands first.
- **F2** (fixed as proposed): card 21's `pauseTick` gains the rule that an entry already past `GraceEnd` at the watcher's first read, not owning the orch, is marked handled in memory with no Esc and no resume line; the table test gains the late-watcher row.
- **F3** (fixed as proposed, plus one consequent change): card 3 creates `SetGateSlug(slug)`, documents it, and gains the `SetGateSlug` row in the Network/ListNetOps test; card 17's Intent has `battenCLI.wire` call `pauserecord.SetGateSlug(slug)`.
  Consequent change: `Gate.HoldsParentLock` becomes `func(slug string) bool`, called with the gate's current slug, and card 23's closure reads whether a parent-lock record names the slug it is given.
  A closure built at install time would otherwise keep checking the prime's empty slug after `SetGateSlug`, so the parent-lock exemption would never apply to the new slug.
- **F4** (fixed as proposed): card 25 lists `internal/shedverbs/doc.go` under **Edit:**, and its Intent states the doc's new pause sentence.
- **F5** (fixed as proposed): card 22's `Status` carries the same network operations list as `PauseReport`, from `deps.NetOps`, with parent-lock marking; the `resume_test.go` Status case asserts an open marker under a parent lock is listed and marked.
- **F6** (fixed as proposed): card 26's `consumeArbiterAttempt` bullet now states the real bound: an older binary's `pause_requested` is carried through Relaunch and honoured at the next producer boundary.
- **F7** (fixed as proposed): card 12's ImpactSummary says seven wiring sites (six wire functions and shuttle's `Command` pre-run).

## Disputed

None.

## Deferred

None.
