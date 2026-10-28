# Fixer report — plan, round 1

## Changes

- F2, fixed as proposed: card 17's Edit list gains `internal/orchengine/doc.go` and `internal/orchengine#Watcher.queueResumeLine` after `internal/creelengine/doc.go`, and its Intent says the orchengine package doc's three reminder passages and `queueResumeLine`'s doc comment are rewritten to say the orch's mail arrives through its hooks, or the bell when idle, once no pause or hold mutes it.
- F3, fixed as proposed: card 15's Intent answers an error `interruptTarget` returns together with true as a store failure after the key, with the existing after-key error.
  The `TestStore_Mail` store-failure row is split: one row closes the journal so `interruptTarget`'s `interrupted` append fails, the other leaves the journal writable and has the interrupt function call `SetStrandsError` (the fake's existing method), so the post's re-read fails; both assert the same error.
- F4, fixed differently: the proposed rows on `fakeEngine` could never read `NoTurn` true, because `fakeEngine` implements no `InputBoxReader` (`internal/shuttleengine/fakes_test.go:347-478` defines no `InputBoxText`), so `readBox` (`internal/shuttleengine/readiness.go:66-70`) finds no box and the planned rule reads a turn as possible.
  Card 7 instead creates `guardedReadinessEngine`, `readinessEngine` plus `InterruptGuard` (`RunningTurn` reading `RUNNING`, `InterruptSettle`), selected by a new `guarded` engine setting beside `styled` in `TestRunner_SessionIdle_ReturnsScriptedClassification`, whose doc comment names it; rows on it pin a draft with no running turn as `NoTurn` true, `Idle` false and a zero `TurnEndedAt`, and the same pane with a running turn as `NoTurn` false.
- F5, fixed as proposed: card 18's interrupt-and-bell scenario interrupts the session's first turn, the one its launch prompt started, so no earlier Stop has run.
  The self-check (proposal lens) kept the clause that a user interrupt fires no Stop beside that premise.
- F6, fixed as proposed: card 15 wraps the unreadable-pause refusal with "; nothing was interrupted or posted" and a way forward naming a post without `--interrupt` or a later `post --interrupt`, and `TestStore_Mail` gains a `SetPausedError` row asserting no key, nothing stored and the post without `--interrupt` named.
- F7, fixed as proposed: card 7 moves `internal/shuttleengine/doc.go` and card 10 moves `pattern/PATTERN-daemon-wakeups.md` from Extends to the end of Edit, and both drop their empty Extends section.
- F8, fixed as proposed: card 13 re-signs `internal/creelcli#hookRead` to `func hookRead(cmd *cobra.Command, engine string) (event shuttleengine.HookEvent, wrote bool, err error)`, and its Intent says `hookReadCmd`'s `RunE` passes the three results to `hookReadExit`.
  The review's `hookCmd` is named `hookReadCmd` (`internal/creelcli/hookread.go:38`).
- F9, fixed as proposed: `TestHookReadExit` keeps only the rows no `TestCreelCLI` row reaches, a Stop with a block written and an acknowledgement error and a Stop with nothing written and an error, and names `TestCreelCLI` as the cover for the rest.
  The self-check (claims lens) restored a tool-boundary call whose claim fails after the parse, which no CLI row reaches.
- F10, fixed as proposed: card 15's no-hooks refusal says "by its typed notice or reminder"; the self-check widened it to "when its engine is unknown or has no hooks".
- F11, fixed as proposed: card 10's `HookWait` refusal names `lym creel read` for a program sender and `lym creel post --wait-reply` for EXTERNAL, and its test row asserts both.
- F12, fixed as proposed: card 12's Intent says `capabilities.go`'s file header names creel's hook delivery in place of tool-boundary context rendering; no target changed.
- F13, fixed as proposed: card 16's Tests paragraph says `prepare_test.go`'s file header names the prompt file carrying the notice and the trust sentence; no target changed.

Changes carried from the parent's recorded decision "A strand with no hook-delivering engine keeps the pane path", which arrived during this round:
card 8 sets `PaneMail` for an empty or unregistered provider, rewrites its Bound, and its `TestHubDelivery` row wants a no-provider strand as `PaneMail` and an unmarked `claude` strand as neither;
card 9 drops `TestSmokeCreelClaudeStrand` from its Edit list, since its hand-added strand stays a pane-mail strand, and rewrites its strand-path sentences;
card 11's no-bell sentence and the overview's second Shared Decision follow;
card 17 keeps `SANDBOX-CREEL-SUITE.md`'s scenarios, whose strands all come from `lym reed add` and so keep the pane path, adding only an intro sentence on hook delivery.

`lym loom validate-plan` answers ok after every change.

## Self-check

- fix-consistency, LOW, card 7's new rows named no test and left its doc comment stale: fixed as proposed, the rows join `TestRunner_SessionIdle_ReturnsScriptedClassification` through a `guarded` engine setting and its doc comment names the guarded engine.
- fix-claims, LOW, card 13's cut `TestHookReadExit` dropped a tool-boundary error after the parse that no CLI row reaches: fixed as proposed, that row is kept.
- fix-affected-set, MEDIUM, card 15's help text still said a Claude strand gets hook delivery, against the pane-path ruling for hand-added strands: fixed as proposed, the Long text names a strand shuttle launched with creel's hooks and a strand whose engine is unknown or has no hooks.
- fix-affected-set, LOW, the same card's refusal named only an engine with no hooks: fixed as proposed, it says "unknown or has no hooks".
- fix-affected-set, LOW, `forwardCmd`'s and `holdCmd`'s Long and `Store.Hold`'s doc comment still describe reminders: fixed as proposed, card 17 adds the three to its Edit list and its Intent rewrites them.
- fix-proposal, LOW, card 18 dropped the fact that a user interrupt fires no Stop, which the bell assertion rests on: fixed as proposed, the clause stands beside the first-turn premise.

## Disputed

None.

## Deferred

None.
