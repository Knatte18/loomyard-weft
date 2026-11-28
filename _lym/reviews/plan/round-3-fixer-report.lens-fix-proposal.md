```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F19, F20 and F21 that the fixer carried out in card 12, and in card 9 for the empty-subject row.

**Findings**

- **LOW**, card 12, the `teardownRecorder` bullet, text "and leave `LivePair` nil".
  The wording says to leave the field alone, but `pairteardown`'s `OriginSeams` always fills `LivePair`.
  So the rows must set it to nil after building the seams, and an implementer who reads "leave" literally keeps the real probe.
  Evidence: `internal/pairteardown/originseams.go:145-160` returns `LivePair: func(slug string) ...` with no nil branch.
  `internal/fabricengine/originreconcilemoves.go:528` skips the probe only when `LivePair` is nil.
  Suggested wording: "and set `LivePair` to nil on the returned seams".

**Proposals checked with no finding**

- **F19, owner-error row:** the card says the `OwnerOf`-error row provokes the real reader's error with board bytes that do not decode. That case reaches `readOriginOwners`. `ReconcileOrigin` calls it (`originreconcile.go:571`) before `readOriginEnded` (`:575`) and before any move. Only `judgeAgainstOrigin`, `recordedPairs` and `recordedRuns` run earlier, and none of them parses origin's board.
- **F21, empty-subject row:** moving this row to card 9's `TestClassifyEntries` holds up. The empty subject falling back to the slug happens in the classification loop: it sets `entry.AdoptSubject = subjects[slug]` and builds the way forward from `entry.AdoptTarget()` (`originreconcile.go:707-709`). Card 9 moves that loop into `fabriccore.ClassifyEntries`, which takes `subjects` as input. The real `OwnerReader` never returns an empty subject on its own, so the row cannot stay in the moved tests.
- **F21, real teardown seam:** the seam's report does carry tags, kept lines and the mutation record (`originseams.go:184-196`). The plan already lists as unverified whether the real `teardownEnded` → `t.Run` runs in tier1 with no tmux server. I found no case that disproves it within my call budget.
- **F20, inline assertions:** asserting inline over `ListFailedPushes` and `Entries` rests on no fact the code disproves.

**Tool calls**

1. Read `originreconcile.go:700-775` and grepped for the empty-subject fallback, to see the owner read and where the subject falls back. My context did not hold the engine's reconcile code.
2. Read `pairteardown/originseams.go:162-240` and grepped for the ledger read order, to see what the real teardown seam reports and when each origin read runs. My context held only up to line 170.
3. Read `originreconcile.go:530-572`, to confirm that nothing parses origin's board before the owner read. My context did not hold the start of `ReconcileOrigin`.
