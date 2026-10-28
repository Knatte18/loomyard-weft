# Fixer-report, code round 1

## Changes

All four findings fixed as proposed, one commit each, gate tests green on `internal/shuttleengine`, `internal/creelengine` and `internal/creelcli`.

- **F1** (`internal/creelcli/cli.go`): the `creel` and `creel post` help now say "Any other strand, a hand-added one or a session launched without those hooks included, gets a typed notice or reminder".
- **F2** (`internal/creelengine/verbs.go`): the `post --interrupt` refusal ends at "which the receiver gets by its typed notice or reminder"; the engine and `lym creel read` clause is gone.
- **F3** (`internal/creelcli/smoke_tmux_test.go`): the doc comment of `TestSmokeCreelFakeProviderPane` opens with the symbol name again and keeps one sentence about the strand.
- **F4** (`internal/shuttleengine/session_test.go`): two rows added to `checkLandingStartConfirmation`, an unreadable box and the sent text after an empty read, both waiting out the window with no key sent.
  The helper gained a `sentText` parameter, because the two callers type different text (`/clear` and `/compact the plan`), so the reviewer's literal `/compact` would have been wrong for one of them.
  The helper's doc comment names the new cases.

## Disputed

None.

## Deferred

None.

## Self-check

All four forks returned a report; each said "no findings".

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
