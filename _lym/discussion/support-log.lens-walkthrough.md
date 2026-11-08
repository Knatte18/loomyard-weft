```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism end to end on one concrete case.

**Findings**

1. **BLOCKING — D8 with D7: in a tapestry run the reviewer will usually get "no current report".**
   - Case: a tapestry run whose Webster block ends with the `webster` key's gate. That gate writes the report against HEAD, H1.
   - The rows between it and Code-Review move HEAD. Webster's fix rows commit after the end gate, and Parent-Align runs only the `parent-align` key, which does not list the step. When the parent has moved, that row can also make a new HEAD, H2.
   - Code-Burler-Review then reviews H2. D8 fills the marker only when the report's commit equals the reviewed HEAD, so the reviewer sees the no-report line.
   - D7's whole purpose, getting findings to Code-Review, is missed on the first review round of a tapestry run. Only darn works on that round: Darn-Write's clean gate comes before verify, and nothing commits between it and Code-Review.
   - Evidence: `contracts/recipes/tapestry-recipe.yaml` row order is Webster, Parent-Align, Drift-Verify, Code-Bouncer, Code-Burler-Review. D7's shipped keys are `webster`, `darn-write` and `code-burler-fix`.
   - Fix options:
     - list the step on `parent-align` (the last gate before review);
     - or make D8 accept a report whose judged tests' function hashes and package file hashes still match HEAD, instead of requiring equal commits.

2. **MEDIUM — D7 step 4: two new tests that duplicate each other are both flagged.**
   - Case: the diff adds A and B, which cover the same new blocks. No map test covers them.
   - Step 4 counts "another test judged in this step" as a cover, so A is contained by B and B by A.
   - Both get findings, and a reviewer who follows both deletes the coverage of the new code.
   - The tool avoids this with the greedy removable set (D1 owns it), but D7 never applies that set to the judged tests.
   - Fix: run the judged tests through D1's greedy pass, or let a judged test count as a cover only for a test it does not mutually depend on, and say in the report which findings can go together.

3. **LOW — D3 rationale vs D7 step 1: D3 says "without running git", but step 1 still needs the merge base's text of each test function.**
   - Reading that text is a git read at the merge base, through `gitrepo` and `gitexec` per `PATTERN-gitrepo-client-boundary`.
   - The claim holds only for comparing blocks. The record should say step 1 reads the base text through the gate's existing diff route.

4. **LOW — D1 acceptance criterion: "the same contained set for the same package and profiles" is ill-defined for the step.**
   - The step judges only the diff's tests, and its covers are the map's tests whose function hash is unchanged.
   - Its set can therefore differ by design from the tool's package view.
   - The criterion should be about the shared verdict function on the same block sets, not the three callers' outputs.

5. **LOW — D5 staleness: a map goes stale only when the package's own Go files change.**
   - Case: a landing changes a dependency so that a map test of `internal/fabricengine` takes another branch.
   - Its package-scoped blocks change, but every hash stays the same, so the map is never rebuilt.
   - This feeds D7's false-positive bound ("a map test lost coverage since the map was built"), but D7 frames that bound only as time since the map was built, not as a gap staleness never closes.
   - Fix: state it as a bound, or key staleness on the package's test-binary dependency set.

**Per decision: the case I used, and whether its purpose is met**

| Decision | Case | Purpose |
|---|---|---|
| D1 | the tool, the verb and the step share the runner and eligibility | met; see finding 4 |
| D2 | the fabricengine profile is instrumented module-wide, then filtered to the package's files; a candidate whose unique block is in `hubforge` shows "not contained module-wide" | met |
| D3 | a block shifted by 3 lines keeps its key; an edited block gets a new key, so the new test counts as unique | met; git claim in finding 3 |
| D4 | written via `internal/state`, read back the same | met |
| D5 | the verb is run in the prime checkout after a landing. Landing refuses a dirty parent code checkout (`internal/landingshed/finalize.go`), so the checkout is updated, files match HEAD and the map is rebuilt | met; see finding 5 |
| D6 | the dobby counts the landing and starts one verb run; a second landing during that run queues one more | met |
| D7 | A and B duplicate each other | broken; see finding 2 |
| D8 | a tapestry run | broken; see finding 1. A darn run's case is met |
| D9 | prose only | met |

**Tool calls**

- `grep` of `internal/landingshed` and `internal/parentlock` for how a landing updates the parent checkout: D5 and D6 rest on the prime checkout being current, and my context did not show that.
- `ls` of recipes and `grep` of motif row names: to learn which rows run after the gate. My context had no recipe row order.
- `grep` of row names and keys in the darn and motif recipes: for the darn row order.
- `sed` of Darn-Write's gates: to check whether a commit happens between the verify gate and Code-Review.
- `grep` of tapestry row names: for the tapestry order between Webster and Code-Burler-Review, which finding 1 rests on.
