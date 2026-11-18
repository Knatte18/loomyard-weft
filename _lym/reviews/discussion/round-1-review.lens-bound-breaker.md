```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guarantee and claim about existing code in the record. Of the claims attacked below, 8 hold and 10 break; each break is filed as a finding.

**Claims attacked**

| Claim | Result |
|---|---|
| D1/D4/D7/D9 study figures: 56 MB, 153 functions at ≥500 tests, four flaky failures reaching none of their landing's files, 85 % unproductive mutants | held: report.md:31, 54, 257, 281-292, 171 |
| D7: the 4-space line is read as the test's output by `gateslot.FailedTests` | held: runlog.go:308-309; failurereport.go:246-253 |
| D7: line counts toward the report cap, and reruns parse unchanged | **broken** (F1) |
| D7: card fixer's report is the call's stderr | held: detachfixer.go:191-201 |
| D7: line content fits the report line | **broken** (F10) |
| D3 rationale: Darn-Write and Code-Burler-Fix gates go through `verifytree.Verify`, which refuses a dirty tree | held: verify.go:326-339; loomshed/verifygate.go:183,217 |
| D3: refresh only when the package has no modified or untracked file, "since the measurement writes no map for such a package" | **broken** (F2) |
| D3/D4: refreshed maps make the note fresh enough for the writer to add missing tests | **broken** for cross-package reach (F3) |
| Open risks: a tapestry writer's `batch` gate runs on uncommitted edits | **broken** for the system-run batch gate (F4) |
| D1: absent or stale-stamped index gives the same answer | **broken** for an added map file (F5) |
| D1: same-size, same-tick rewrite bound | held, and stated |
| D2: unchanged text with moved lines is not changed | **broken** across files (F6) |
| D5: report used only when it matches HEAD | **broken** for a red-tier report (F7) |
| D9/D11: per-package fresh map available, and `lym gate mutate` replays a diff | **broken** in the prime (F8) |
| D3 bound: "inside a gate slot" | **broken** for `slots: all` tiers (F9); held on Bolt, where no tier sets `slots` |
| D9: block lines come from the block table, and the mutants can be filtered per line | held: profile.go:15-22 (`ProfileBlock` Start/EndLine); mutant.go:133-155 (`Mutant.File`/`Line`/`lastLine`) |
| D12: config load fills a template key a tuned `gates` entry lacks | held: configengine/doc.go:8; gateslot/doc.go:113; Bolt's gate.yaml:43-51 sets no `reports` |
| D2/support log: the old finder went with the selector in `72b26d1a` / `b92c95ee` | held: git log |
| D3: no refresh writes the hub's shared maps; D6 writes nothing; D8 `--since` and the 200-commit cap; D10 `mutation` refused without `reach` | held: no counterexample found |

**Findings**

- **F1 — BLOCKING · design · decision-record.md D7 ("It counts toward the report's line cap…", `gateslot.FailedTests` "parse the report unchanged", and the D7 parse acceptance criterion).**
  - Counterexample: a red tier whose failure report runs to 78 lines without triage, with five failed tests. The triage lines push it past `failureReportLines = 80` (failurereport.go:29), so `renderFailureReport` appends the `... N more lines cut` line (failurereport.go:212-215).
  - `FailedTests` then hits that line in its default branch: `reportRunTier` fails and it returns `ok=false` (runlog.go:310-315; its doc says "a cut report … give false").
  - The rerun path in loomshed/verifygate.go:248 and landingshed/verifygate.go:309 then gets no named tests where it got them before. So the stated "same failed tests" guarantee is false near the cap.
  - Evidence: failurereport.go:29, 194-215; runlog.go:293-316.
- **F2 — MEDIUM · design · decision-record.md D3 ("only when it has no modified or untracked file, since the measurement writes no map for such a package"; the D3 acceptance criterion).**
  - The `--map` write also refuses a package when a *linked* package owns a modified or untracked file (PATTERN-test-speed.md:305; gatecli/testmap.go:585).
  - Counterexample: a hand-run gate with uncommitted edits in `internal/a`, and a clean changed package `internal/b` that links `a`. D3 treats `b` as refreshable, so it spends the build and the per-test runs of `reach_budget_min` and writes no map.
  - The note then gives no "uncommitted files" reason for `b`, and D9 judges `b` "not judged" with a wrong cause.
- **F3 — MEDIUM · design · decision-record.md D3/D4 (refresh only "the packages that hold" changed functions; the unreached list).**
  - Reach from another package comes from that other package's map (D1), which D3 never refreshes; hub maps predate the diff.
  - Counterexample: a new function `a.F`, called and tested only through `b.TestX`. `b`'s hub map has no `a.F` key, so D4 lists `a.F` as reached by no mapped test, and D9 lists no possible killer from another package.
  - D7 states this bound for triage ("a stored map may lack a function the run added"), but D3 and D4 do not, while D3's rationale claims the note lets the writer add the missing tests.
- **F4 — LOW · consistency · decision-record.md Open risks (tapestry `batch` bullet) and the D3 rationale.**
  - The weaver's batch gate runs through `verifytree.Verify` and fails a dirty tree with `UncommittedFindings` (weaverengine/strand.go:149-165), so the system-run `batch` gate never sees uncommitted edits. The risk holds only for a writer's hand-run `lym gate test --gate batch`.
  - The D3 rationale names only Darn-Write and Code-Burler-Fix as dirty-safe, leaving `batch` out.
- **F5 — LOW · design · decision-record.md D1 (index "stamped with each map file's size and modification time"; "an absent or stale-stamped index gives the same answer").**
  - A stamp check over the index's own listed files misses a map file the index never listed.
  - Counterexample: a prime `--map` run writes a new package's map and then ends early because HEAD moved ("a moved HEAD ends the walk", PATTERN-test-speed.md:307), before its index write. Every stamp in the old index still matches, so readers miss the new package.
  - D1 must stamp the directory's file set, or bound this case.
- **F6 — LOW · design · decision-record.md D2 / the D2 acceptance criterion ("unchanged but whose lines moved … not found").**
  - `FuncKey` includes the file (blockkey.go:131-138). A function moved verbatim to another file of its package gets a new key: D2 finds it "new", and D4 lists it as unreached.
  - Several `init` functions in one file share one key, so a text comparison between them is ambiguous.
- **F7 — LOW · design · decision-record.md D5.**
  - The redundancy review note rejects a report written on a red tier: it "never matches" (PATTERN-test-speed.md:335).
  - D5's match is on changed functions and file hashes only. So a D3 report that judged nothing on a red tier, followed by a green rerun with no file change, still matches HEAD, and the reviewer gets an empty note instead of the "no current report, and why" line.
- **F8 — LOW · design · decision-record.md D9/D11.**
  - In the prime, D3 refreshes nothing and `--map` writes the hub's maps, not anchor maps (PATTERN-test-speed.md:222). So no package has a "fresh" map by D9's test, and `lym gate mutate` or the `mutation` step in the prime reports every package not judged. The record does not say so.
- **F9 — LOW · design · decision-record.md D3 bound ("inside a gate slot").**
  - The `--map` measurement holds the larger of tier0's and tier1's slot asks, `slots: all` ranking highest (PATTERN-test-speed.md:302).
  - On a hub whose tier asks `all`, every darn-write gate's reach step blocks the hub's gates for up to `reach_budget_min`; D3's cost bound omits this. On Bolt no tier sets `slots`, so the claim holds there.
- **F10 — NIT · design · decision-record.md D7/D8 ("ends with this command for the test").**
  - Report lines are cut at `failureReportLineBytes = 300` (failurereport.go:31, 331-339).
  - Five fully qualified function keys plus the trailing `lym gate reach --test <pkg>.<Test> --since <last green commit>` can pass 300 bytes, and the cut drops the command, which is the line's actionable part.

**Tool calls**

1. `gateslot/config.go` fill grep and Bolt `gate.yaml` slots: needed for the D12 fill claim and the D3 slot bound.
2. `git log` of `72b26d1a`/`b92c95ee` and weaverengine/strand.go:140-165: needed for the D2 commit claim and whether `batch` runs through Verify.
3. configengine grep for the template-key fill and tier slot asks: the per-entry fill code wasn't in context.
