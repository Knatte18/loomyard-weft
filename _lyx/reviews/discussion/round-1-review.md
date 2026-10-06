---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:Decision 9
    summary: Decision 9 passes a list of in-scope packages to -pkg, but the base tool takes one go pattern and has no section splicing, so the rerun and the scoped replacement are not specified as the tool can do them
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:Decisions 2, 3, 5
    summary: Decisions 2, 3 and 5 say "stated reason" without saying where it lives, and a move-then-delete leaves no durable record of which covering test took the assertion
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:Decision 10 and first acceptance criterion
    summary: The keep criterion is not checkable through the base tool, which ignores the directive, and a rerun after deletions can list new candidates with no stated stop
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:Decisions 1 and 7
    summary: No rule for a required reference update or deletion that lands in a core file or internal/testkit, while the last acceptance criterion demands no dangling reference
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:Decision 12
    summary: The coverage method does not say whether -coverpkg is used, and its rule allows explained drops while its rationale says any drop is a mistake
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:Decision 6
    summary: Decision 6 judges a helper unused without naming the tags, so a helper used only by an llm file could be deleted
---

### [MEDIUM] Decision 9 asks the tool for what the base tool cannot do

**Location:** `decision-record.md`, Decision 9.
**Issue:** At this base, `cmd/testtiming` takes `-pkg` as one string and passes it as a single argument to `go list` and `go test` (`runRedundancy`, `listPackages`, `runPackageTimings`).
The in-scope set is "everything but six packages and `internal/testkit/...`", which one go pattern cannot express, and a space-separated list fails as one argument.
The tool also rewrites `-out` whole; the section-splicing write is core's D3, not present here.
Decision 9 says "its package sections replace those packages' sections", which is right only if the replacement is a manual or scripted splice from the scratch file, and says so nowhere.
Running `./...` into the scratch path would also re-measure the core packages for nothing.
**Fix:** State that the rerun is one invocation per in-scope package (or per group a single pattern covers), each to its own scratch file.
State that the splice into `docs/benchmarks/test-redundancy.md` is done by this task outside the tool, replacing each in-scope `## <package>` section with the scratch section of the same name.

### [MEDIUM] Reason location and a silent move-then-delete

**Location:** `decision-record.md`, Decisions 2, 3 and 5; acceptance criterion on deleted candidates.
**Issue:** Decision 2 and Decision 3 keep a candidate "with a stated reason" and Decision 5 leaves a test in place "with that reason stated", but only Decision 10 says where a reason lives, and it covers listed candidates only.
A tier move not made (Decision 5) is not a candidate, so its reason has no stated home beyond "the hub is told".
Decision 2's move-then-delete path leaves nothing durable: the deleted test is gone, and the criterion "its unique assertions live on in a covering test" cannot be checked by a reviewer without a record of which test received what.
**Fix:** Each of the three decisions points to where its reason lives: the `//testtiming:keep` directive for a kept candidate, the PR body for an unmade tier move.
Decision 2 adds that the PR body lists, per deleted candidate whose assertion moved, the receiving covering test.
That list bounds the path: a deletion with unique assertions and no listed receiver is not allowed.

### [MEDIUM] Keep criterion against a tool that ignores the directive, and an unbounded rerun

**Location:** `decision-record.md`, Decision 10 and the first acceptance criterion.
**Issue:** The record gives the spelling `//testtiming:keep <reason>` and "directly above its `func Test…` line", so the plan writer can produce it without reading core.
At this base the tool lists every candidate regardless of the directive, so the criterion is checkable only as "every test the rerun lists has the directive", by comparing the rerun's lists with the source, and the record does not say so.
A deletion changes the covering sets of the remaining tests, so a rerun after pruning can list a candidate that was not listed before.
Decision 2 says such a candidate with no unique assertion is deleted, while the criterion says every listed candidate carries a reason naming what it pins; the record does not say how many passes run or when they stop.
**Fix:** Say the criterion is checked by comparing the final rerun's listed candidates with the directives in the source.
Say the prune repeats rerun, then decide per Decision 2, until the rerun lists only candidates that carry a directive naming a unique pinned assertion; a test with nothing to pin is deleted or folded, never annotated.

### [MEDIUM] Boundary against core when a change needs a core edit

**Location:** `decision-record.md`, Decisions 1 and 7; last acceptance criterion.
**Issue:** Decision 1 forbids edits under the six core directories and `internal/testkit/`, and keeps in-scope files that name a core file unchanged unless an in-scope change requires an edit.
Decision 7 requires updating every reference to a deleted or renamed test, and the last criterion says no doc, comment or stencil names one.
If such a reference sits in a core file or a testkit file, the two rules collide and the record does not say which wins.
**Fix:** Add to Decision 1 that a reference or helper dependency that would need a core or testkit edit is left alone, listed in the PR body for the hub, and exempted from the last criterion.
Add that the in-scope change itself is then made so it does not break core: a test that a core file names is kept, not deleted or renamed.

### [LOW] Coverage method and its rationale disagree

**Location:** `decision-record.md`, Decision 12.
**Issue:** The report's "covered by construction" holds under the module-wide `-coverpkg` the tool uses, while Decision 12 takes per-package `-coverprofile` and does not say whether `-coverpkg` is set.
Without it, code covered in another package only through a deleted test is not seen.
The rule also lets a drop be "explained by deleted or moved tests" while the rationale says a `Removable: yes` deletion leaves every block covered, so any drop is a mistake; both cannot hold.
Decisions 2 and 4 are consistent with Decision 12 as a block-level check: it neither promises nor claims to catch a lost assertion, which the open risks already say.
Decision 8 is consistent too: a test that passes alone is not a candidate, so it is neither deleted nor treated as covering.
**Fix:** State the `-coverpkg` choice, and state that a drop is explained only by a deletion of a `Removable: yes` candidate whose blocks the report shows covered elsewhere, otherwise it is restored.

### [LOW] Unused helper judged under one tag set

**Location:** `decision-record.md`, Decision 6.
**Issue:** The task measures under `integration,tmux` and excludes `llm`.
A helper used only by an `llm` file looks unused under those tags, and deleting it would break the `llm` build; the `go vet -tags llm` criterion would catch it late.
**Fix:** Decision 6 says a helper is unused only when no file under any tag, `llm` included, calls it.

## Checked and not a finding

- `-redundancy`, `-tags`, `-pkg` and `-out` all exist at the base; `-tags integration,tmux` in timing mode prints per-package elapsed, test count and serial time, so Decision 11 holds.
- The report has one `## <package>` section per package, core and `internal/testkit/...` among them, so a section-scoped replacement is well defined.
- The `go vet` tag matrix and the `llm` exclusion match the Decisions and the criteria.
