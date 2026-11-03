```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-1 fix agrees with the sections around it in the decision record and support log, and find any change whose twin elsewhere was left as it was.

- MEDIUM — decision-record.md, Acceptance criteria, the new D2 bullet: "a viewer on the lower-index overview window that ends moves to the other overview window, never the shell."
  "Ends" also covers a replacement, and the bullet then contradicts two sibling statements.
  The Constraints say "a replacement that ends the viewed overview window selects its new window".
  The D1 criterion says "a replacement that ends the viewed overview window selects its new window".
  Read literally, the new bullet says that a replaced lower-index overview window sends the viewer to the other overview window, while D1 and the Constraints say the viewer goes to the replacement window.
  Evidence: `ReplaceStrand` calls `selectStrandWindowLocked(strand.PaneID)` when `endedCurrent && replacingWithOverview` (`internal/reedengine/replace.go:97-98`).
  The ending fallback, `selectRelevantWindowIfCurrentLocked`, applies only on other paths such as a remove or a reconcile.
  The bullet should be limited to a window that ends without a replacement, for example when its strand is removed or its pane exits.

- LOW — decision-record.md, D5 Bound: the old line "An older lym process still doing reed ops refuses a second overview add of its own" was replaced by the old-build driver sentences.
  The support log still records the earlier version.
  Turn 4 reads "existing-paths 3 (an older live lym process, dropped key): fixed; D5's bound names both."
  The bound still names a dropped key and an older process, but the "older live lym process" that line describes, which only refuses an add, is gone.
  The support log's "Review rounds" section still reads "_No rounds yet._" and records nothing of this round.
  A reader comparing the ledger with the record would notice that Turn 4 no longer matches the bound text.
  This is LOW only if the round record belongs to the loop owner rather than the fixer.

I found no other disagreements.
- Q5 now agrees with the Auto-mode assumptions and with Open risks "None.".
- The new D5 sentences agree with D2's rationale because they qualify it as a bound.
- They agree with D3 and D4, which they name as the ways back.
- They agree with D5's body ("re-entrant and keeps a live driver").

Tool calls: none; my context already held every file this lens needed.
