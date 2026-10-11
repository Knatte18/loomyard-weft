---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/loomengine/template.yaml:34
    summary: the template lost its trailing newline when fix_start was removed, so the retired-key row and the selfreport row of TestLoadConfig_Loads append their key onto the last comment line and no longer test anything
  - id: F2
    severity: NIT
    class: consistency
    location: internal/shuttleengine/doc.go:73
    summary: the doc repeats the two deferral constants as literals (45 seconds, 20 minutes) that hold.go already declares
---

### [MEDIUM] Template without a trailing newline makes two config rows vacuous

**Location:** `internal/loomengine/template.yaml:34`, `internal/loomengine/config_test.go:143` and `:180`

**Issue:**
Card 7 removed the template's last line, `fix_start: parallel`.
The edit left `review_max_bounces: 3  # ...` as the final line with no `\n` (the diff shows `\ No newline at end of file`; `od -c` of the file's tail ends on `escalation`).
`TestLoadConfig_Loads` builds two rows as `ConfigTemplate() + "<key>: <value>\n"`: `selfreport: false` and the new `fix_start: after-review` row.
With no newline in the template, each appended key lands on the same line as the trailing `# ...` comment and is part of that comment.
Neither key reaches the YAML parser, so both rows load the template defaults and pass whatever the loader does with a real retired key.
The `selfreport` row passed meaningfully before this diff and has lost its coverage; card 7's `fix_start` row, which must show a hub `loom.yaml` still carrying `fix_start` loads to the template defaults, never exercises it.
The missing newline also leaves the seeded template without a final newline for `lyx config reconcile --apply`, which card 7 relies on to append and remove keys; I did not run that path.

**Fix:**
Restore the trailing newline at the end of `internal/loomengine/template.yaml`, then confirm both rows fail if their key is made invalid (for example `fix_start` rows still load, and the `selfreport: false` row reads `false`).

### [NIT] Doc pins constants that hold.go declares

**Location:** `internal/shuttleengine/doc.go:73`

**Issue:**
The held-turn paragraph writes `holdNoticeSettle (45 seconds)` and `holdNoticeMaxDeferral (20 minutes)`.
`hold.go` declares both values once; the doc copy goes stale silently if a constant changes (`scribe:prose`, "Don't pin a perishable specific").

**Fix:**
Drop the parenthesised values and name the constants alone.

## Judgments on the focus questions

None of these is a finding.

- **Cards 3 and 4, `internal/seatengine`.**
  The per-advisor gate (`seatGate`), `ForkSubagents`, and `Round` pass through `seatSpec`, `startSeat` and `probeSeat` as the cards state, and `validateSeatRoles` refuses the three chair and non-required misuses.
  A required advisor's unaccepted end reaches `stepEnder.end` on every path: non-done, gate failed, wait error and `Accept` error in `seatRun.watch`, a start failure in `Engine.Run`, and a not-found advisor in `Engine.Resume`.
  `Engine.join` stops the sender and the chair, waits for its result, stops every advisor, and sets `EndedStep` on that advisor.
  The one case where `EndedStep` stays false on a fired step is a chair that ended in the same instant, which the card assigns to the chair's end.
  `stopAdvisors` waits on each advisor's goroutine, so an `Accept` marker write cannot outlive the attempt.
- **Card 8 `decideRound`.**
  The reviewer-or-fixer choice reads `reviewer.EndedStep`, with the start-failure test ahead of it by `StartError`; the fixer rules read the chair's outcome and the recorded `released` flag, as the card specifies.
  The chair's wait error is handled in `concludeRound` rather than `decideRound`, with the same wrapped error and the reviewer-ended case still deferred to `decideRound`.
  The round's outcome, review files, fixer-report and gate rules are carried over unchanged.
- **Cards 1 and 5.**
  `settleDeferredHold` calls the predicate only after `holdNoticeSettle`, ignores it after `holdNoticeMaxDeferral`, and the pending turn end is dropped when a new event or turn start is read.
  `advisorWork.outstanding` counts an advisor only while its watch goroutine is running, and an unreadable state counts as idle.
  The chair's notice still goes through `notifyHeld`, so a stalled chair reaches the parent.
- **Card 8 `LiveRound`, `Resume`, `ErrHalfNotStopped`.**
  `LiveRound` gains one unexported pointer, so it stays comparable; `ErrHalfNotStopped` is wrapped together with `ErrSeatNotStopped` (`%w: %w`) and `shedadapters` is untouched in the diff.
- **Cards 7, 10, 11, 12.**
  A grep over the tree finds no remaining `FixStart`, `RunnerShuttle` of burlerengine, `awaitReview`, `roundHalves`, `stopHalf`, `probeHalf`, `burlerReviewRole`, `burlerFixRole`, `halfSpec`, `reviewerEnd` or `fixerEnd`; the only `await-review` text is the stencil `mustNot` claim.
  Docs and `doc.go` changes land with the code.
  The affected packages' tests pass under `lyx gate test`.
- **Test economy.**
  The new seat-engine tests are the table-driven ones the cards name, and no new top-level test duplicates an existing one; I found no added behavior without a test.
  The one test that fails to test its behavior is the `fix_start` row in F1.
