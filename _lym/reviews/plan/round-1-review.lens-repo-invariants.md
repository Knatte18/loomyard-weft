```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. For each card I checked test seams, doc text the card falsifies, and whether the card can pass its own gate.

**Per card, each invariant checked**
- Card 1:
  - Test seams: held. There is no reflective seam test, and `TestFabricReplay` only gains rows.
  - Doc: held. boardengine `doc.go` is listed.
  - Order: held.
- Card 2:
  - Test seams: held.
  - Doc: **broken**. The file header of `holdwrite.go` still says `WriteHold` is the only writer of `holds`.
  - Order: held. `WriteAdopt` has no production caller until card 9, but `cmd/lym` is outside card 2's card and batch scope.
- Card 3: test seams, doc and order held. fabricengine `doc.go` and the `AdoptScratchDir` and `DropLocal` docs are listed.
- Card 4: test seams, doc and order held.
  - The codex and claude per-row tables do not demand a case for every row.
  - `norecoverydeny_test.go` reads its forms from the table.
  - The shellguard `doc.go` enumeration stays true.
- Card 5: test seams, doc and order held.
  - The dobbyengine, orchengine, orchcli and beamcli `doc.go` files are listed.
  - The orphaned adoptrecord watch functions are only caught by `cmd/lym`, which is outside card 5's scope.
- Card 6: test seams, doc and order held. beamcli `doc.go` is listed.
- Card 7: test seams, doc and order held.
- Card 8: test seams, doc and order held.
  - The import from adoptcli through tendercli and orchcli makes no cycle: tendercli depends on orchcli, never on adoptcli.
  - `fixtries.Write` keeps the `ResetTries` caller in `deps.go` until card 9.
- Card 9:
  - Test seams: held.
  - Doc: **broken**. Line 3 of `PATTERN-dobby-runs.md` is left.
  - Order: **broken**. The card's tier-1 batch gate reaches the test-only guard in `cmd/lym`.
- Card 10: test seams, doc and order held.
- Card 11: held. `tenderengine/doc.go:61` ("An adopt pause is an operator pause") stays true for legacy entries, so it is not a finding.

**Findings**

1. **MEDIUM, design.** Card 9 versus card 10: card 9's batch gate fails the test-only guard unless Webster batches card 10 with it.
   - Card 9 edits `cmd/lym#unboundedWaitFiles`, which puts `cmd/lym` in its card's packages.
   - The batch gate runs tiers [0, 1] over those packages, and `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` is tier 1 (`cmd/lym/testonlycode_linux_test.go:82-83`).
   - After card 9, no package imports `internal/adoptrecord`, so every function in it is reached by no main package until card 10 deletes it.
   - The overview's Shared Decision says the guard "runs at the plan-level gate, after card 9", but by the plan spec's verify model (the batch gate runs over the cards' packages) it can run at card 9's batch.
   - Fix: fold card 10's package removal into card 9, or move the `cmd/lym` edit out of card 9.

2. **MEDIUM, consistency.** Card 9 leaves `internal/adoptcli#parentWarning` unreachable, and no card deletes it.
   - Its only caller is `resumeSlug` (`internal/adoptcli/adopt.go:505`), which card 9 deletes.
   - `parentWarning` is in neither card 9's Delete list nor its Uses.
   - The test-only guard (`cmd/lym/testonlycode_linux_test.go:94`) reports it whatever the batching.
   - The constant `workerOutName` (`caller.go:33`) is left unused too, since its users `spawnWorker` and `follow.go` go. It is a constant, not a function, so the guard does not catch it.
   - Fix: add `internal/adoptcli#parentWarning` and `internal/adoptcli#workerOutName` to card 9's Delete list.

3. **LOW, consistency.** Card 9's pattern edit leaves the opening summary of `PATTERN-dobby-runs.md` false.
   - Line 3 still reads "an adopt pause lifts only by hand" (`pattern/PATTERN-dobby-runs.md:3`).
   - The card names only the "Adopt pause" section (lines 19–21), the `--no-orch` exception and `PATTERN.md`'s overview line.
   - Fix: card 9's Intent also rewrites line 3 to the declared PATTERN text.

4. **LOW, consistency.** Card 2 leaves the file header of `holdwrite.go` false.
   - It reads "holds the adopt hold's setter, the only writer of board.json's `holds` list" (`internal/boardengine/holdwrite.go:1-3`).
   - Card 2 adds `WriteAdopt`, `FixHold` and `ClearHold` as writers of `holds`, but edits only `doc.go` and the `WriteHold` member.
   - Fix: add `internal/boardengine/holdwrite.go` to card 2, or put the new writers in that file and reword the header.

**Tool calls**
1. grep for `reflect.` in the touched packages' tests, and for adopt wording in `PATTERN.md`, `pattern/` and `contracts/specs`: to find reflective seam tests and pattern text the manifest did not cover.
2. grep for `reflect.` uses in adoptcli, beamcli and orchengine tests, for `doc.go` files naming the adopt record, pause or worker, and for `lym adopt` in contracts and plugins: to tell seam-coverage tests from value comparisons and find docs no card lists.
3. grep for adopt in the tenderengine, fronttask, lymdirs, battencli and hubgeom `doc.go` files, and read `orch-role.md:88-95`: to judge whether docs no card lists are falsified.
4. `go list -deps` of tendercli and orchcli, plus the codex `guards_run_test.go` head: to check for an import cycle in card 8 and whether the per-row tests demand a case for every row.
5. grep of `cmd/lym/norecoverydeny_test.go`: to check whether the no-denied-recovery test pins the form list that card 4's new form extends.
