```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency: checking that the fix's changed text agrees with its sibling cards, rows and cross-references, and finding any twin of a change that was left as it was.

- **LOW**: card 9, the new `internal/hubreconcile/doc.go` paragraph (lines 62-63), together with card 2's F5 exception text.
  Summary: the fix rewrites doc.go's lines 3 and 33 and changes PATTERN-config-layers lines 7 and 77-78. It leaves the twin sentence in doc.go itself unchanged.
  Evidence: `internal/hubreconcile/doc.go:21-22` says "The walk removes no key the running binary's template lacks … The operator removes such a key with `lym config reconcile --apply` or `lym config &lt;module&gt; --unset &lt;key&gt;`". A moved old key is a key the template lacks, and card 6's walk removes it from the hub base files by renaming or dropping it. Card 9 now owns doc.go as an **Edit** target but rewrites only lines 3 and 33. Cards 6 and 8 only extend doc.go. So no card adds the move-table exception that card 2 now adds to PATTERN lines 7 and 77-78.
- **LOW**: card 8 at line 5 (the signature), lines 33, 39 and 41 (`beforeCommit`) and line 49 (`beforeRunSettingsCarryCommit`).
  Summary: the rewrite says `RewriteRunSettings` commits nothing itself, because the detached sync commits and pushes, and that the hook runs as `afterSave`, before the render. The parameter and hook names still say the hook runs "before the commit".
  Evidence: line 39 says "`beforeCommit`, which it passes as `boardCriticalSection`'s `afterSave`". The rewritten paragraph says "Only when some entry changed does it save and render, and the board's detached sync commits and pushes". Line 49 keeps `beforeRunSettingsCarryCommit`.

Tool calls:
- One grep of `_lym/plan/0*.md` and `_lym/plan/1*.md` for "start verb", `beforeCommit`, `beforeRunSettingsCarryCommit` and "template lacks". My context showed only the cards the fixer read, so I could not see the other cards' twins of the changed text.
