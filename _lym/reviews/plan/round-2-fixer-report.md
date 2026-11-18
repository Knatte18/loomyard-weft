---
dispositions:
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed-differently
    reason: "The proposal has a done relaunch on the start path return Result{} and nil after relaunchUnderAccount has already finalized it, but finalize is not idempotent (internal/shuttleengine/wait.go:1978) and Wait has no early return on a terminal record (internal/shuttleengine/wait.go:268-308), so Wait would finalize the run a second time. relaunchUnderAccount instead takes a startPath flag and, with it set, finalizes none of its died or timeout ends itself: it ends them as abandonLoad does, through classifyDeadlineExpiry and abandonStartup."
  - id: F16
    disposition: fixed
---

# Round 2 fixer-report — plan

## Changes

- **F14 (fixed).** Card 5 drops `internal/battencli#Command` from Edit.
  Its Wiring paragraph now has `battenCLI.wire` fill `pickAccount` beside its other seams, with the closure that builds the new pair's location, takes `hubgeom.EngineWorktree` of it and calls `PickAccount(keepEntry)`.
  It states that every existing test that calls `wire` and drives `CreateWorktree` past the add, or into its `complete` branch, reaches the real pick, which writes nothing on a hub with fewer than two accounts.
  `TestWire_CreateWorktreePicksTheNewWorktreesAccount` asserts `wire` leaves the field non-nil, then overrides it after `wire` as the owner-seam tests override `owner`.
- **F15 (fixed differently).** Card 14's `relaunchUnderAccount` signature gains `startPath bool`.
  `Wait` passes false and `loadSkillsThenPrompt` passes true.
  The load-turn paragraph states the start path's return.
  A relaunch whose pane starts returns `Result{}` and nil.
  With every output file present (`classifyDeadlineExpiry`), a died or timeout end clears the mark, leaves the strand and returns `Result{}` and nil, unfinalized, so `Wait` ends the run done once.
  Otherwise the end goes through `abandonStartup`, which finalizes once, removes the strand and returns an error wrapping `ErrNotStarted`.
  A stop-ended relaunch returns the stop's outcome beside an error wrapping `ErrNotStarted`.
  Step 4 says each died end of steps 2 to 4 and the startup timeout go that way with `startPath` set.
  Uses gains `Run.abandonStartup` and `Run.classifyDeadlineExpiry`.
  A new row drives `Start` with no start retries over a failing second `Prepare`: `Start` returns no run and an error wrapping `ErrNotStarted`, and the run is finalized died once with its strand removed.
  The same row with every output file present returns the run, and its `Wait` ends it done with one session end.
  Card 16's start-path call passes true and states it follows card 14's start-path return, and its `retryAPIError` call passes false.
  The proposal was not taken as written for the reason the frontmatter gives.
- **F16 (fixed).** Cards 14 and 16 move `internal/shuttleengine/doc.go` from Extends to Edit and name the sentences each rewrites:
  - card 14: the usage-limit paragraph, the re-send paragraph, and the skill-load hold paragraph;
  - card 16: the skill-load hold paragraph and the re-send paragraph.

## Self-check

- fix-consistency, LOW, the tripwire row did not name the start-path ends: fixed. The row now names the start path's `classifyDeadlineExpiry` and `abandonStartup` ends among the audited sites.
- fix-consistency, LOW, card 14's ImpactSummary was stale: fixed. It now names the start-path return and the doc.go rewrites.
- fix-claims, LOW, card 5's claim about the tests that reach the pick was too broad: fixed. It now covers only tests that drive `CreateWorktree` past the add or into the `complete` branch.
- fix-claims, LOW, card 14 stated the start-path timeout in step 3: fixed. The sentence moved to step 4 and covers step 2 to 4's died ends and step 4's timeout.
- fix-proposal, MEDIUM, the re-send rewrites missed doc.go's "A run with no retries waits such a turn end out to its deadline": fixed. Cards 14 and 16 each name that sentence and the exception it gains.
- fix-proposal, LOW, `loadSkillsThenPrompt`'s doc comment twin of the hold paragraph was unnamed: fixed. Cards 14 and 16 each name that sentence and what it gains.
- fix-affected-set: no findings.

`lym loom validate-plan` passes with informational findings only, before and after the self-check edits.

## Disputed

None.

## Deferred

None.

## Rulings

None.
