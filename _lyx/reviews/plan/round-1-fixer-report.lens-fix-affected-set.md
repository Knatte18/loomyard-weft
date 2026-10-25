```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. I reran the set searches for F1, F3 and F8 against the change set.

Unchecked findings: F2, F4, F5, F6, F7 and F9. None of them rests on a set of sites, so there was no search to rerun.

**F1: covered.** Four production sites read `landingshed.HandsBack`: `arbiterstop.go:107`, `arm.go:308`, `drivewiring.go:320` and `battencli/wire.go:975`. Card 14 now covers all four: `Check`'s `LandingHalt` sits beside the new `DecisionHalt`, and `specFor`, `goDriverDepsFor` and `battenCLI.wire` all move to `arbiterstop.HandsBack`.

**F8: covered for `parentreview`.** No other file under `pattern/` or in `PATTERN.md` names `parentreview`. `pattern/PATTERN-shuttle-provider-seam.md:23` still lists "alignment" among the role settings, but that belongs to card 6 and is outside F8's premise.

1. **MEDIUM.** Card 16's widened closing bullet ("test fixtures that hold the row name `Merge-In` or the gate key `merge-in` as a string literal take `Parent-Align` and `parent-align`") reaches test members that its `**Edit:**` list does not name.
   The rerun search finds old-row-name literals outside every listed or renamed member:
   - `internal/battenshed/innerrun_halted_test.go:49` (`TestInnerRun_HaltedStatesWaitExemptWithoutSpawning`)
   - `internal/battenshed/classify_test.go:48,55,110,125,126` (`TestClassifyRecords`)
   - `internal/loomcli/finalizeguard_test.go:47` (`TestFinalizeDirtGuard`)
   - `internal/loomcli/settledguard_test.go:27-28` (`TestSettledSegmentGuard`)
   - `internal/loomcli/reviewstatus_test.go:451` (`TestParentLockWaiting`)
   - `internal/loomcli/landingdeps_test.go:157-164` (`TestRowEntry`)

   It also finds one old-gate-key literal: `internal/gateslot/runlog_test.go:27` (`GateKey: "merge-in"`).
   The bullet tells the implementer to change these literals, but their test members are missing from the edit list, so either the list misses part of the set the bullet claims or the bullet claims more than the card covers.
   The fix should either add those members under `**Edit:**` or narrow the bullet to the fixtures the row's gate key and producer identity read.
   The members that already reach the bullet need no change: `handback_test.go` is covered by `TestHandsBack`, `entries_simple_test.go` by `simpleEntryCases`, `recipe_test.go` by `TestEntryNumbers`, `darnwrite_test.go` by `TestDarnWrite_Call`, and `mergein_test.go` and `mergein_fixer_test.go` by the rename.

Tool calls:
- grep for `landingshed.HandsBack` in production Go: the context held only partial wiring output, and I needed F1's full site set.
- grep for `parentreview` under `pattern/` and `PATTERN.md`: the context held only `PATTERN-cwd-resolution.md`, and I needed F8's full set.
- grep for `"Merge-In"` test literals: the fixer searched only `"merge-in"`, so the row-name half of F3's set was unsearched.
- awk for the enclosing test functions of the new hits, plus the second PATTERN file's line: the context lacked the symbol names needed to check them against card 16's edit list.
