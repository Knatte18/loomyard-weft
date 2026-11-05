```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that each set-premise finding's fix covers every member of its set, not only the ones the review named.

Unchecked findings: F8. Its premise is one method's behaviour, not a set, so I ran no search for it.

**F3 (MEDIUM): one site in the old-flow set is still not covered**
- Changed text: card 5's and card 7's added F3 bullets, which reach `runTakesNoSlot`, `loomrecipe/doc.go` and `dobbyengine/doc.go` but not this member.
- Summary: `beamForkPrecondition`'s doc still says the landing run holds the beam "from Parent-Align on", and no card says to rewrite it.
- Evidence: `internal/battencli/beamfork.go:48` reads "A beam whose board entry beamEntry is held by its own landing run, from Parent-Align on, names the landing…". Card 7 re-signs `beamForkPrecondition`, but its Intent only adds the ready refusal. Card 5 already gives the stale `boardengine/runlock.go:49` text to its `RunHolds` doc bullet. Every other hit for "from Parent-Align" / "Parent-Align on" is covered by a card or is tapestry/darn text that stays true.

**F2 (LOW): a doc that card 1 edits still blurs the two retired-row mechanisms**
- Changed text: card 1's added split sentence.
- Summary: card 1 edits `internal/shedengine/doc.go`, whose `RenameProducers` paragraph talks about "each retired name". Card 1 now adds a second retired-row meaning to the same package (`Shed.Retired`), but its Intent does not say that doc keeps the two apart, as card 5 now requires of `loomrecipe/doc.go` and `loomcli/doc.go`.
- Evidence: `internal/shedengine/doc.go:128` reads "It rewrites each retired name in current_producer, entered_row, …". Card 1's Edit list includes `internal/shedengine/doc.go`.

Sets confirmed fully covered:
- F1/F4: `ResolvePullRequest(…, true)` appears only in cards 14 and 15, and both are fixed. `PullRequestMode` is read only by `prClose` and refusal 5.
- F7: rows naming a PR-Review hold appear only in cards 5 and 10, and both are fixed.
- F5: card 15's refusal 1 already reads "running or halted". Cards 13 and 14 rely on card 12's wording.
- The `lym beam reopen` text sites are left to card 6, whose Edit list I saw only in part. It deletes the reopen symbols.

Tool calls:
1. Grep for old-flow phrases across `internal`, `contracts`, `PATTERN.md` and `docs`: to rerun F3's search, since my context held only the sites the review cited.
2. Grep for `RetiredRows` / retired-row wording in `internal` and `docs`: to rerun F2's search for the same reason.
3. Read card 6's head plus the `beamfork.go` and `runlock.go` docs, and grepped which cards name `beamForkPrecondition` and `docs/overview.md`: to check whether the extra hits from call 1 are covered. Neither the card nor those code spans were in my context.
