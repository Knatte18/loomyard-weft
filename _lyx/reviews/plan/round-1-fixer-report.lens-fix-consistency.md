```yaml
lens: fix-consistency
outcome: reported
```

Findings (fix-consistency):

1. MEDIUM — card 26, Tests sentence: "a row error carrying `OwnedElsewhereError` leaves the pass's exit at the driver envelope's own code, not 77", added as a `TestPassExitCode` row.
   The row can't be written as a `TestPassExitCode` row, because `passExitCode(err)` maps any error carrying `OwnedElsewhereError` to 77.
   Card 26's own Intent says the codes are set "only for a refusal before the step loop", and it is `driveRun` that applies that rule, not `passExitCode`.
   So the row belongs at the `driveRun` level, as `TestBattenRun_RefusesWhileAPauseCoversTheSlug` drives it through `clihelp.Execute`, or the sentence has to say which test drives a row error.

2. LOW — card 31, the `dobbyReading` sentence: "an undecodable record reads the same with one Warn, as card 4's `ReadState` reads it as absent".
   Card 4 now has `ReadState` itself warn once on an undecodable record.
   If `dobbyReading` adds its own Warn, the same cause gets two Warns.
   The sentence should say the Warn is `ReadState`'s.

3. LOW — card 25's ImpactSummary still reads "Batten's driver deps gain the arbiter seam, its open-attempts read and its loop's consume hook".
   The fix added the halted-row `GotoGuard` to `loadedArbiterDeps`, with its own test row, and the summary doesn't mention it.

No contradictions with sibling cards for the other changes:
- The prime-anchor working directory agrees across cards 4, 27, 28, 33, 37 and 40.
- The `dobbyengineAllowedImports` chain agrees across cards 4, 27, 35 and 36.
- The draft release texts agree across card 7's refusal family.
- Card 24 now refuses two hand-backs, and no other card mentions a batten landing halt.
- The `ResumeInWindow` doc twins agree between cards 28, 37 and 39.
- The two `PATTERN-cwd-resolution` additions agree between cards 4 and 22.

Tool calls:
- One grep across the plan cards for twins of the changed phrases (no-goto-guard, landing halt, `ResumeInWindow`, hub root, fresh-start and left-pair texts, revive text, `TestArbiterSpec`). My context showed only the cards I had read.
- One read of `internal/loomcli/arbiter.go:374-395`, `internal/battencli/cli.go:258-264`, and the revive and pause mentions in cards 16 and 42. I needed how a `GotoGuard` refusal is wrapped, the `batten goto` argument form, and whether any sibling card repeats the changed notice texts.
