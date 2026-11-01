```yaml
lens: fix-affected-set
outcome: reported
```

Task: re-run each set-shaped finding's search (F3, F7, F11, plus F15 and F20 already re-checked by the fixer) and check that the fix covers every member of the set.

Unchecked: F1 (single module, not a set), F2 (one allowlist), F15 and F20 (the fixer already re-ran the same grep and `go list -deps` that I would have), F19 (row cut, no search).

Findings:

1. LOW. Card 13 (old card 12), the widened `TestBoltReplay_NamesNoBoardFile` bullet: the list of extra functions misses `Bolt.logReplay`.
   - F11's fix asked for every function in the card's Edit list that lives in `pull.go`, `pushclassified.go`, `originreconcilemoves.go`, `originreconcile.go` and `bolt.go`.
   - `Bolt.logReplay` is in `bolt.go` (grep for `func (b *Bolt) logReplay` matches `internal/fabricengine/bolt.go`), and the card lists it under Edit as `internal/fabricengine#Bolt.logReplay`.
   - The guard's named set is `Fabric.weftEndedByOrigin`, `goneEntryEnded`, `readOriginEnded`, `addLocalCreatedStamps`, `readOriginOwners` and `Bolt.endedByOrigin`, so `logReplay`'s body goes unscanned.
   - The impact is small: it logs and holds no board file literal today. The current board-literal grep finds only `replay.go`.

2. LOW. Card 7 (new): the sentence "`hubforge`'s `Hub.BoardDir` … stays" understates where that accessor reaches the Bolt checkout.
   - `internal/hubforge/seed.go:58-67` writes `configengine.ConfigFile(h.BoardDir(), "fabric")` and commits there.
   - `internal/hubforge/template.go:163` calls `commitRelocatedBinding(tb, hub.BoardDir())`.
   - Both are production (non-`_test`) Bolt-checkout uses inside hubforge, not only test callers.
   - The exclusion is stated, so this is not a gap in the set. But the card's claim that the accessor's "callers' tests being card 6's" does not cover these non-test helpers. A reader auditing D9 would find them unaddressed.

Covered sets:
- F3: every test directory the plan names now has a planned `TestMain`: `internal/entrymerge` in card 11, `internal/heddleengine` in card 14 (which also covers the render, seam and replayer tests), `internal/heddlecli` in card 19. No other new test package exists.
- F7: no `BoardDirName` use remains outside fabric in production code. The only other Bolt-checkout reach is `Hub.BoardDir()`, covered in finding 2.

Tool calls:
1. Grep for `BoardDirName` and `BoardDir()` outside fabricengine: to re-run F7's search for spellings other than `fabricengine.BoardDir(`, which my context had not searched.
2. Check every test directory the plan names for a `TestMain` on disk or in the plan: to re-run F3's set across all cards, since my context had checked only the three named packages.
3. Find which files hold card 13's Edit members, and which non-test files hold board literals: to re-run F11's set, since my context had not located `Bolt.logReplay`, `OwnerReader` or the literal-holding files.
