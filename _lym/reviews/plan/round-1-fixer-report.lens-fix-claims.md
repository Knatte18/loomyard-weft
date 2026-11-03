```yaml
lens: fix-claims
outcome: reported
```

Task: check that the factual claims the fixer added or rewrote hold against the code at HEAD.

**Findings**

1. **MEDIUM.** Changed text: card 2, the new rename-target cleanup bullet ("A failure of that removal is returned as a failed move is"), and card 5's new paragraph (the error "names the record file left at the old slug's name").
   Summary: the cleanup path runs only when the old file is absent, so the move's error text ("the record stays at %s until the entry's next Discussion approval...") and card 5's help claim are both false there. No record is left at the old name.
   Evidence: card 2's move error text is `rename moved the entry but not its decision record: %w: the record stays at %s ...`. The cleanup bullet applies "When `decisionFileName(from)` is absent". Card 5 says the error "names the record file left at the old slug's name".

2. **MEDIUM.** Changed text: card 2's removal bullet, "so `Board.RemoveTask`, `Board.Prune` and `Board.MergeTasks` keep their contract that an error means nothing was saved".
   Summary: no such contract exists at HEAD. `boardCriticalSection` can already return an error after `store.Save()`.
   Evidence: `internal/boardengine/board.go:152-164`. After the save, an `afterSave` error is returned (`:157-158`), and so is a render failure (`return nil, fmt.Errorf("render: %w", err)`, `:162-163`).

**Claims checked and found to hold**
- Card 8: the copy runs only after `deps.commit()` succeeded or `afterFailedCommit` returned landed, because a failed commit without an amendment refuses at `decision.go:783-784`.
- Card 8: `hasPlan` and `rowWritten` parse the same seed recipe first and refuse an unparseable one (`decision.go:688-698`, `:909-923`), and `RetiredRows` cannot fail (`recipekeys.go:156`).
- Card 3: the `board` module's `CheckLoad` is `boardengine.LoadConfig` (`configreg.go:130`), and `newLoadFailure` passes when the load after the write succeeds (`configcli.go:290-293`).
- Card 5: `boardcli/doc.go:29` holds the `front`/`on_land` sentence.

**Tool calls**
- Read `internal/loomcli/decision.go:720-934`, to check card 8's claims about when the copy runs and about the upstream recipe parse (the post-commit path and `preRun` were not in context).
- Grepped `RetiredRows`, the board `CheckLoad` and `committedLacksEntry`, to check whether `pastApproval` has an error source besides the parse and to confirm card 3's loader.
