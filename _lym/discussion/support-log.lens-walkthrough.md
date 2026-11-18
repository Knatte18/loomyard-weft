```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's stated purpose.

**Findings**

1. **MEDIUM — D3 (and D9/D10 downstream).** The fresh-map refresh can't measure a dirty package, so the writer's note misses its own new functions.
   - Case: a tapestry implementer edits `internal/foo/bar.go`, adds `NewThing`, and runs its `batch` gate before committing.
   - Step: D3 hands the package to the `--map` measurement. `testmaptree.go`'s `dirtyOwnedFile` refuses a map for any package with modified or untracked owned files, which lists it as `unmeasured`.
   - Result: D3 falls back to the hub map, which doesn't know `NewThing`. The note lists it as unreached even when the writer's new test reaches it. That misses the operator's stated purpose ("the writer adds the missing tests itself").
   - The record says the fallback happens only "past the budget". It should also cover a dirty package, say so in the note, and either require a clean tree or state the bound.

2. **MEDIUM — D5.** The match key ignores test files, so a stale "unreached" note still matches HEAD.
   - Case: the darn writer reads D3's note "`Foo` reached by no test", adds `TestFoo` in `foo_test.go`, and commits. The next gate call keeps or rewrites the report, but take the case where the review fills from a report written before the test commit.
   - Step: D5 matches on "HEAD's changed functions and their text hashes". D2 leaves test files out, so the changed-function set and hashes are identical.
   - Result: the report matches and the review gets "`Foo` unreached", a false candidate. The redundancy report matches on test hashes and package Go-file hashes, test files included.
   - Fix: D5 should also match on the Go-file hashes, test files included, of the packages whose maps the report read, as `RedundancyReport.matches` does.

3. **MEDIUM — D8.** First-parent commits of the current branch don't hold the parent's landings that matter.
   - Case 1: the run's gate fails on `TestX`. The D7 line says "reaches none", and the fixer runs the printed command from the run's worktree.
   - Step: "first-parent commits of the current branch after `<rev>`" walks the run's own commits, then the parent's history up to the merge base.
   - Case 2: after Parent-Align merges the parent in, the landings since the merge base sit on the merge commit's second parent. They are invisible, and the merge commit's diff against its first parent lumps every parent landing into one entry.
   - Result: the purpose, "names the landings whose changes the test reaches" for a parent branch gone red, is missed from a run worktree.
   - Fix: D8 should walk the parent branch, read from git or lym's records per `PATTERN-code-conventions`, or take an explicit rev to walk. The current branch is only right when the orch runs it on the parent.

4. **MEDIUM — D11.** `lym gate mutate` names no map refresh, so on a fresh worktree every changed block reads as uncovered.
   - Case: the trial runner checks out a landed commit and runs `lym gate mutate --base <rev>`.
   - Step: D9 runs mutants against "the package's fresh map". D11 says only "runs the check of D9", and nothing builds that map. The worktree has no maps of its own, and the hub map lacks the diff's new block keys.
   - Result: every changed block reads as uncovered and no mutant runs. The trial counts zero survivors and wrongly concludes the check doesn't pay.
   - Fix: state that the verb first refreshes the changed packages' worktree maps as D3 does, and under which slot and budget.

5. **MEDIUM — D9/D10.** A package answered from the hub map gets false "uncovered" blocks.
   - Case: D3 leaves package `P` past `reach_budget_min`, or dirty as in finding 1, and answers it from the hub map. D10 then plans mutants for `P`.
   - Step: changed blocks map through HEAD's coverage-binary block table to block keys. Edited blocks have new text hashes, so the hub map's table doesn't hold them.
   - Result: every changed block in `P` is reported "uncovered, no mutant". The review reads that as a coverage gap that may not exist.
   - Fix: D10 should skip such a package with a line saying it had no fresh map, rather than report its blocks as uncovered.

6. **LOW — D7/D8.** The appended D8 command can't be complete, because `--since` is required and the gate doesn't know the last green.
   - The record should say the line prints `--since <last green commit>` as a placeholder the caller fills.

7. **LOW — D7.** Triage lines count against the failure report's 80-line cap (`failureReportLines`).
   - Their order relative to failure output isn't stated, so on a many-failure report they can push out later tests' got/want lines that a fixer needs.
   - The record should place each line beside its test's own lines, or cap the triage lines.

**Decisions walked, case used, purpose met or not**

- **D1** (`Foo` in `P`, reached by `P.TestA` and by `Q.TestB` via linked functions; worktree map of `P` present) — met.
- **D2** (a function moved down 10 lines with unchanged text; a body edit; a new function) — met.
- **D3** (an implementer's `batch` gate on uncommitted edits) — not met, finding 1.
- **D4** (a function reached by 600 tests) — met, through the cap of five plus the count.
- **D5** (a test added after the note) — not met, finding 2.
- **D6** (`lym gate reach internal/x/y.go#Func` on an unmapped function) — met.
- **D7** (a flaky `TestRunRelay` failing in a run that doesn't reach it) — met for the line itself; findings 6 and 7.
- **D8** (from a run worktree after Parent-Align) — not met, finding 3.
- **D9** (a package with a fresh map, changed `if` condition) — met. With a hub-map fallback, not met: finding 5.
- **D10** (the key lists `mutation` without `reach`) — the refusal is met. The fallback case fails, finding 5.
- **D11** (a trial on a new worktree) — not met, finding 4.
- **D12** — met. The tuned hub `gates` entries get the template's `reports` filled in at config load (`gateslot/doc.go`: "Config load fills a template key a tuned `gates` entry lacks"), so the template change reaches this hub.

**Tool calls**

1. Grepped the report validation in `internal/gateslot/config.go` and the dirty-tree refusal in `internal/gatecli/testmaptree.go`: my context lacked whether a map build refuses a dirty package.
2. Read `decodeGates` and the `gates` layering lines of `gateslot/doc.go`: my context lacked whether the template's `reports` reach a hub whose own `gate.yaml` lists none.
