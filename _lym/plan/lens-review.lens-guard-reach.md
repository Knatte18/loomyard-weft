```yaml
lens: guard-reach
outcome: reported
```

The plan has two blocking guard-reach defects:

- **D12 / card 13:** the guard for a lost baseline at Drift-Expect never fires in the case D12 names.
- **stop-prefixes / cards 8 and 13:** if `DeferHaltPrefix` opens a reason, the stop chain stops handing it to the parent.

Five lesser findings follow.

1. **BLOCKING — D12 / card 13 (Drift-Expect): the missing-baseline guard is unreachable when `.lym` is cleaned or on another machine.**
   - Card 13 raises the `lym drift defer` Stuck only for "a counted capture whose baseline directory is missing".
   - "Counted" is `CountedCapture`, which needs `capture.json` to be found (`internal/driftengine/records.go:207-209`). `capture.json` is ephemeral (`paths.go:59`, under `.lym/drift`).
   - In the scenarios D12 names (`.lym` cleaned, another machine), `capture.json` is gone with the baseline. Neither guard state can occur.
   - Drift-Expect then falls through to writing the listing over a missing baseline directory. Card 3's `ListCaptures` errors on that, and nothing turns the error into the defer Stuck D12 requires.
   - Card 13's test row "a missing baseline directory is a Stuck naming `lym drift defer`" passes only because its fixture keeps `capture.json`.
   - Fix: judge "counted" at Drift-Expect from the tracked `scope.json` `awaited_at`, not `capture.json`, and add a test row with no capture record.
   - Card 10 step 3 and card 16 step 3 meet the same state as "the capture never counted". They still name `lym drift defer`, but the reason is wrong for a capture that did count.

2. **BLOCKING — Shared Decision stop-prefixes / cards 8, 13: an opening `DeferHaltPrefix` makes `DecisionHaltPrefix` undetectable.**
   - `arbiterstop.HandsBack` tests `strings.HasPrefix(reason, shedadapters.DecisionHaltPrefix)` (`internal/arbiterstop/arbiterstop.go:62-63`).
   - Card 8 says `DeferHaltPrefix` "opens every reason … whose way forward is `lym drift defer`". Card 13's Stuck opens "with `DeferHaltPrefix` and `shedadapters.DecisionHaltPrefix`", in that order.
   - Cards 16 and 19 instead put the decision prefix first ("after an optional decision-halt prefix").
   - Built as cards 8 and 13 word it, every defer Stuck at Drift-Expect, Drift-Verify and Drift-Recheck loses its decision-halt handback, so the arbiter or tackler becomes eligible for a stop only the parent may settle.
   - Fix: fix the order once in the Shared Decision (`DecisionHaltPrefix`, then `DeferHaltPrefix`) and make cards 8 and 13 say it.

3. **MEDIUM — D6 / cards 8, 16: the `--discard` fault for an `expected.yaml` with no seal commit can never fire at Drift-Verify or Drift-Recheck.**
   - Card 16 step 6 records `Unsealed` for any run past Drift-Expect with no seal commit, before step 7's `Check`.
   - Card 8's `CheckSeal` returns `Empty` on `Unsealed` (second branch), ahead of the "`expected.yaml` with no seal commit: Fault naming `--discard`" branch.
   - The row path therefore never yields the D6 criterion "one with a file and no seal commit fails naming `--discard`". Only the gate entry and `lym drift check`, run before any Drift-Verify, can.
   - Card 19's "`--discard` clears … an unsealed file" is then never needed at a row.
   - Fix: either check the file-without-seal fault before honouring `Unsealed`, or set `Unsealed` only when no `expected.yaml` exists. Add a `TestDriftVerify` row reaching the fault through the row.

4. **MEDIUM — D3 / card 13: an in-flight run with no recorded `diff` is unguarded at Drift-Expect.**
   - D3 says a scope record with no recorded `diff` "records config's values at its next drift row".
   - Card 13's Drift-Expect never records `Diff` or `KeyPatterns`. It uses "the recorded mode" for the listing and for `NewExpectationGate`.
   - A run seeded before this lands and parked before Drift-Expect reaches it with `Diff` empty, so `ParseExpectation(…, mode "")` and the listing choice have no defined branch. Only card 16 (Drift-Verify step 2) records it.
   - Fix: Drift-Expect records config's `diff` and `key_patterns` through `WriteRecords` when the record has none, with a test row.

5. **LOW — D11 / card 19: `Amend`'s admission covers more rows than D11 admits.**
   - Card 19 admits "a blocked row past Drift-Expect up to and including Drift-Recheck". D11 names "a Stuck writer, Drift-Verify or Drift-Recheck".
   - In tapestry that span also holds Parent-Align, Code-Bouncer, Code-Burler-Review and Code-Burler-Fix (`contracts/recipes/tapestry-recipe.yaml:318-479`), so an amendment is admitted at a blocked Parent-Align or Code-Burler-Fix, where no drift check stopped the run.
   - No card test refuses those rows.

6. **LOW — D15 / cards 9, 26: "refused before any slot is taken" conflicts with card 9's order.**
   - Card 26 puts the `{skip_file}` refusal in `renderCommand`, which `runStep` calls (`steprun.go:86`).
   - Card 9 moves `acquireSlot` ahead of the lock in `RunCapture` and `CaptureCurrent`, so `runStep` runs after the slot is held.
   - In `RunCapture` the refusal also comes after the running record is written and `backUpBaseline` has moved the existing baseline aside (`capture.go:84-90`).
   - The stated guard state ("before any slot is taken") cannot occur on that path. The refusal should run before `acquireSlot` and before the backup.

7. **LOW — D5 / card 13: on an in-flight run, `NewExpectationGate` has no fallback when no mode is recorded.**
   - Card 10's `Check` falls back to config's `diff` when the record has none. Card 13's gate reads only "the recorded mode".
   - On the in-flight run of finding 4, the gate's parse mode is undefined. It is the same root cause and should get the same fix.

Tool calls beyond my context:
- Grepped `runlevel.go` for completed-plan handling and `shedrecipe` for `gateFindingsPath`, to see whether a re-entered Webster short-circuits and where findings files go.
- Read the `GateFindingsFile:` wiring in `loomcli/wiring.go` and `arbiterstop.HandsBack`, to confirm the findings file stays out of the judge's checkouts and that the decision prefix is matched with `HasPrefix`.
- Read `shedrun.GateFindingsFile` and grepped `runlevel.go` for batch-done handling, to confirm the findings path is the run scratch dir and to look for a Webster re-entry short-circuit (none found).
