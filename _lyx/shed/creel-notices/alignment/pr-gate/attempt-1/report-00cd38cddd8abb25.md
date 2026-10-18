---
verdict: APPROVED
---

Alignment at `pr-gate` for `creel-notices`, pull request 720 at `0fd6a9e24652e1b92d1437d9357f6a0861f38b0d` against `main` (`7acbf58c2`, the merge base).

## Goal and decisions

The diff carries the board entry's goal: notices and every line lyx types into the orch go through creel, the watcher's notice queue and its typing idle probe go, and the reminder carries no content.
Spot checks against the decision record:

- D1: the stage 1 withholding is gone, `inbox` loses its withheld keys, and the fold still reads `withheld` records and acts on none (`internal/creelengine/fold.go`, `journal.go`).
- D2: `composeReminder` (`internal/creelengine/reminder.go`) names count, ids and senders only, with `--from <owner>` under a hold.
- D3: `internal/creelengine/command.go` holds the closed neutral set; `InjectCommand` refuses a non-program sender and a non-orch receiver; the CLI has no command-inject flag, and `lyx creel log` shows `command` and `argument`.
- D4, D5, D10: the `Creel` seam in `internal/orchengine/creel.go` has post, mailbox read, inject, outcome, journal lookup, hold, release and withdraw, wired in `internal/orchcli/creelseam.go` as `ORCHWATCH`; step injects carry a ten-minute TTL; the handoff stencils ask for `lyx creel post --reply <id>` answers.
- D5 scope item: `Withdraw` is program-only, Go-client-only, acts by name on an unlisted receiver, and journals `withdrawn`; `shuttleengine.Event` gains its time from `ParseSessionSignals`.
- D6: `NotifyPrime` takes the caller's program name, tries a non-orch strand claim first and retries as the program only on `ErrStrandClaimOutsidePane`, which the Go client returns for the typed `Response.Code`; batten, loom, burler, webster and hibernate call it with their own program names.
- D7: tender checks its own pending revive through `log` and injects as `TENDER` with a one-minute TTL.
- D8: no production code writes `.lyx/notices.log` or `notices/`; `lyx orch cleanup` lists and removes both.
- D9: `SessionIdle` and the guarded interrupt survive only in the pause handler (`internal/orchengine/pause.go`).
- PATTERN: `PATTERN-orch-pane-single-writer` names creel as the only writer and its background file holds the decision record's exceptions; the durable-state background file carries the operator's sentence, shortened as the auto-mode assumption records.

The carried-over open risks (Discussion-Review round 4, Plan-Review round 3, Webster-Review round 1) are design findings the review segments own; none of them makes the change leave the goal or break a recorded decision.

## Notes, not findings

- The `PATTERN.md` index line for `PATTERN-durable-vs-ephemeral-state` reads "nothing a run depends on" where the decision record's PATTERN changes say "nothing a run, the orch or a landing depends on"; the final commit trimmed it for the gate's headroom, and the full sentence stands in the background file, so the invariant is unchanged.
- The branch rewords several unrelated `PATTERN.md` index lines (`told-geometry`, `mutation-record`, `shed-recipe-registry`, `recipe-setting-lookup`, `creel-store`, `friction-capture`, `completion-signal`, `wait-mark-display-only`, `spawn-observability`, `documentation-lifecycle`) to fit the same headroom; each restates its background file, and none changes an invariant.
  Against `main`, where `95be34cf0` shortened some of those lines differently, this reads as a text-only overlap already taken in at the Merge-In (`25863a0b6`).
- `docs/overview.md`'s creel line says the notifying modules sign "as their own program"; D6 also signs as the caller's non-orch strand when creel accepts the claim, so the sentence simplifies that rule.

## Overlap

`lyx loom peers` lists no peer run in flight and no parent commit (`own_row` 22), so there is no overlap and no landing order to give.
`cmd/lyx/testdata/test-budget.yaml`, the `docs/overview.md` module lines and the new `PATTERN-orch-pane-single-writer` background link are shared bookkeeping.

## Board

`creel-agents` (stage 3) keeps every item this run's Scope Out leaves to it; `test-budget-remove` and `batten-start-speed` depend on this task and are not superseded.
The folded-in note `creel-reminder-no-subject` is no longer on the board.
No board entry is made obsolete.
