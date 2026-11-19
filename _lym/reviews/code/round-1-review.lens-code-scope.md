```yaml
lens: code-scope
outcome: reported
```

Task: trace each change in the diff to the board entry's goal points, under the code-scope lens.

**Decisions** (the board entry's `## Goal`; the record has no `Added after Discussion` entries):
- **G1.** For an entry labelled `bug`, a gate step runs the tests the diff adds or changes, named by `testreach`, once on the merge-base tree. It fails when every one of them passes there.
- **G2.** The fix commit carries a `Root-Cause:` trailer, which Code-Review checks like `Loosened-Check:`.
- **G3.** A bug with no seam for a test is a stop the record names, never a silent pass.

**Changes and what each traces to:**
- **C1** `internal/testreach` (`ChangedTests`, `declaredFuncsWhere`/`directoryImportPath` refactor, doc, test) → G1, which says "named by testreach".
- **C2** `internal/gateslot` (`RedFirstCall`, the two exit codes, `PackagesFailedOutsideTests`, doc, test) → G1.
- **C3** `internal/gatecli` (the `red-first` verb, `cli.go` registration, doc, unit and integration tests) → G1.
- **C4** `internal/gatecli/mutate.go` (`diffBase` becomes the shared `verbDiffBase`) → forced by C3.
- **C5** `internal/loomshed` (`NewRedFirstGate`, `BugLabel`, `NoTestSeamTitle`, doc, test) → G1 and G3.
- **C6** `internal/shedrecipe` (the `red-first` vocabulary entry, the `EntryLabels` and `RedFirstCommand` seams, doc, tests, fixture) → G1. The matching edits in `internal/testkit/envkit` and the `internal/battencli` wire-seams test are forced by C6.
- **C7** `internal/loomcli/wiring.go` (`entryLabels`, seam wiring) → G1.
- **C8** `contracts/recipes/darn-recipe.yaml` (the gate row) → G1. The `internal/loomrecipe` budget test edit is forced by C8.
- **C9** `contracts/stencils/darn/darn-template-write.md` ("A bug entry" section) → G1, G2 and G3.
- **C10** `contracts/stencils/loom/loom-rubric-code-review.md` ("Root cause of a bug fix" bullet) → G2.
- **C11** `docs/overview.md` → forced by C3 and C8.

No change lacks a trace, so there is no unrequested scope.

**Decisions and the changes that build them:**
- **G1** → C1, C2, C3, C5, C6, C7 and C8. Partly built: only the darn recipe's Darn-Write row carries the gate (finding 1).
- **G2** → C9 and C10. Built.
- **G3** → C5 (the `No test seam` pass), C9, and the way-forward text in C3 and C5. Partly built: a non-Go hub passes silently (finding 2).

**Findings**

1. **MEDIUM, scope** — `contracts/recipes/darn-recipe.yaml:140`. G1 asks for the gate on any entry labelled `bug`, but only Darn-Write carries it. A tapestry or sampler run of a `bug` entry hands off with no red-first check and no named stop.
   - Evidence: the board entry under `## Goal` has no recipe qualifier (`_lym/discussion/decision-record.md`, `## Idea`). The gate row exists only in the darn recipe (diff `001-contracts-recipes.diff`; no other recipe hunk in the range).
   - The change description admits it ("Only Darn-Write carries the gate … tapestry and sampler runs of such an entry are not checked", `_lym/landing/summary.md`, Design decisions). No `Added after Discussion` entry records that narrowing.
   - Fix: add the gate to the write rows of the other loom recipes, or have the parent record the narrowing with `lym loom decision add`.

2. **MEDIUM, scope** — `internal/gatecli/redfirst.go:119–121` (`runRedFirst`, the `!cfg.HasGoCoverageMaps()` branch). G3 forbids a silent pass, but on a hub whose scoper is not `go` the verb exits 0 with only a note. `NewRedFirstGate` reads exit 0 as a pass (`internal/loomshed/redfirstgate.go:104–105`), so every `bug` fix on such a hub passes the gate with no test run and no record naming a stop.
   - Evidence: diff `006-internal-gatecli.diff:246–247`; diff `010-internal-loomshed.diff:103–105`; the summary's "A hub whose scoper is not `go` judges nothing and exits 0 with a note".
   - Fix: give "judged nothing" its own exit code, which the gate treats as a named, terminal stop, or record the exemption as a decision.

Tool calls: none; the inherited context held the record, the board entry, the diff pieces and the summary.
