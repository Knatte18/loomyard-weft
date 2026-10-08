---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D5, and D2's Bound)
    summary: D5 removes caller coverage for every reference inside a nested module, including callers of a root-module subject and all callers of a subject declared in a nested module, states no bound, and contradicts D2's claim that a package-level caller never weakens
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D9)
    summary: D9 makes a finding whose obvious fix a check blocks not a legitimate finding, so a real defect whose correct fix has a different shape is suppressed, and D9 states no bound on what that lets through
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (D1)
    summary: D1 drops interface-dispatched calls of a re-signed method from caller-uncovered entirely, where today they surface informationally, without stating what still catches the break
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md (D6)
    summary: D6 derives nested modules from Create and Delete targets only, so a Rename that moves a go.mod gives a verdict that differs before and after the renaming card, against D6's own stability rationale
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (Open risks)
    summary: Two requirements live only in Open risks, deduplicating references by file and line (D1) and not reading a flag value such as -o ./bin as a package argument (D8), instead of in the Decisions they bind
  - id: F6
    severity: NIT
    class: decision
    location: _lyx/discussion/decision-record.md (D8)
    summary: D8 does not say which directory a go -C path is relative to when a preceding cd segment set the working directory
---

### [BLOCKING] D5 removes nested-module caller coverage with no stated bound

**Location:** `decision-record.md`, D5; D2's `Bound:` paragraph.

**Issue:** D5 skips every directory below the worktree root that holds its own `go.mod`.
That is a guard removed, and two kinds of caller lose coverage:

- A nested module can hold a caller of a root-module subject.
  A nested module whose `go.mod` requires the root module (typically through a `replace ../` directive, as tools and example modules do) imports root packages by their import path.
  Today's scan tokenizes those files and reports a package-level caller there as blocking.
  Under D5 such a caller is never seen, and nothing else catches the break: D7 keeps the root `go build ./... && go test ./...`, which never builds a nested module, and D7 adds a nested-module command only for a card that targets that module.
- A subject declared inside a nested module (outside `testdata` and the other skipped names) loses every caller, its own package's bare-identifier references included.
  D3 explicitly derives the import path of such a subject from "the nearest `go.mod` at or above its directory", so the design expects nested-module subjects, yet D5 removes the files where their callers live.
  The D1 load from the root cannot type-check those directories either, so both halves of the scan see nothing.

D5's rationale ("a nested module's packages are not part of the root module, and `go/packages` loading from the root never sees them") explains why the loader cannot see them; it is no reason no caller can live there.
D2's bound, "A package-level caller never weakens, because D3 resolves it exactly", is false for both cases above, since today's scan reports them blocking.
In this repository the only nested module sits under `testdata` and is skipped already, but the check is generic and runs on any repository lyx drives.

**Fix:** Either keep scanning nested modules on the fallback path (D3's import-path rule already handles a nested-module file that imports a root package, and a nested-module subject resolves inside its own module), or keep D5 and state in it what it lets through (callers in nested modules, of root and nested subjects alike) and what bounds that, and correct D2's bound to match.
An acceptance criterion pins whichever is picked.

### [BLOCKING] D9's rule can suppress a real defect

**Location:** `decision-record.md`, D9.

**Issue:** D9 says a finding whose fix changes targets or a verify command "must name a fix every check ... admits, and a fix such a check blocks is not a legitimate finding".
Read as written, the finding itself becomes illegitimate when the reviewer's fix is blocked, so a reviewer who spots a real defect (an uncovered caller, a missing verify package) and reaches first for a fix shape a check blocks (the `redundant-file-target` pairing of issue #460) is told to drop the finding rather than find an admissible fix.
The gate's checks did not catch that defect, or the reviewer would not be raising it, so nothing else catches it.
Issue #460 asks that the reviewer not propose a blocked fix; it does not ask that the defect go unreported.
D9 tightens what counts as a legitimate finding without stating what that lets through or what bounds it, which the rubric's attack-surface rule requires.

**Fix:** Bind the fix, not the finding: a finding stays legitimate, its proposed fix must be one every check in the validation-checks section admits, and when the reviewer sees no admissible fix it reports the defect and names the check that constrains the fix.
State that bound in D9, and adjust the matching acceptance criterion.

### [LOW] D1 drops interface-dispatched calls without a stated bound

**Location:** `decision-record.md`, D1; acceptance criterion "A call through an interface whose method set holds `M` is not reported for `T.M`".

**Issue:** Today a `.M` call through an interface yields an informational `caller-uncovered` entry for a re-signed `T.M`.
D1 makes it yield nothing.
Scope's Out list names the case, but D1 does not say what still catches the resulting break: a re-signed `T.M` stops satisfying the interface, which fails to compile at the assignment sites and is caught by the card gate's root `go build ./...`, and only for root-module code.
Under the attack-surface rule, the Decision that downgrades a guard states what bounds it.

**Fix:** Add a `Bound:` to D1 stating that interface-dispatched calls are no longer reported and that the build step of the card gate catches the assignment-site break a re-signed method causes.

### [LOW] D6 ignores a Rename that moves a go.mod

**Location:** `decision-record.md`, D6.

**Issue:** D6 builds the nested-module set from `go.mod` files on disk, plus `Create` targets, minus `Delete` targets, through the card.
A card's Rename group can move a `go.mod` (moving a fixture module is plausible); `planparser.CardTargetDirs` already reads Rename pairs.
For such a plan the old directory counts as a nested module and the new one does not until the rename lands on disk, and the reverse afterwards, so D7's card gate and D8's verdict both differ before and after the renaming card.
D6's rationale ("makes the verdict the same before and after the creating card runs") holds for Create only.

**Fix:** Have D6 treat a Rename pair whose New side is a `go.mod` as a Create of it and the Old side as a Delete, or state in D6 that a renamed `go.mod` is out of scope and add it to Scope's Out list.

### [LOW] Requirements stated only in Open risks

**Location:** `decision-record.md`, `## Open risks`, bullets three and four.

**Issue:** "the reference set must deduplicate by file and line" (from `go/packages` reporting non-test files in both the package and its test variant) and "a flag whose value starts with `./` (such as `-o ./bin`) must not be read as a package argument" are requirements the plan writer must build, but they appear only as risks.
Per the completeness-before-leanness test, a requirement belongs in the Decision it binds.

**Fix:** Move the deduplication requirement into D1 and the flag-value rule into D8, leaving in Open risks only what remains uncertain.

### [NIT] D8 leaves the go -C base directory after a cd unstated

**Location:** `decision-record.md`, D8.

**Issue:** D8 lists the working directory as "the directory a preceding literal `cd <dir>` segment ... set, or ... `go -C <dir>`'s directory".
When both occur, `go -C` is relative to the directory `cd` set; D8 does not say so, so the plan writer could resolve `-C` against the root.

**Fix:** State in D8 that `go -C <dir>` resolves against the directory any preceding `cd` segment set, and against the root otherwise.

## Directives settled without a finding

- D2 and D4: apart from F1, the fallback keeps a package-level caller blocking: D3 resolves it by import path, closing today's alias and dot-import gaps, and a method falls to informational only where type information is missing, which D2 states.
- D3 against D1 and the acceptance criteria: import-path derivation, alias, dot import and the no-`go.mod` case agree with the criteria; no criterion contradicts a Decision.
- D6, D7 and D8: apart from F4, the card form and the whole-plan form use one derivation and agree, and the criterion that a `Verify:` stays silent when the creating card comes later matches D6's "through that card".
- D8's bound: the stated misses (variables, `bash -c`, subshell, import-path arguments) let a command through only to the run-time gate, which D8 names; the check reads plan text and file existence and blocks no command that would pass at run time, apart from the ambiguity in F6.
- D11: the `CLAUDE.md` line adds an obligation and opens no edge or guard, so it needs no bound; not a defect.
- Relocation: beyond F5, Auto-mode assumptions and Notes for the plan writer hold no requirement missing from Decisions or Constraints, and no deliberation narrative in `decision-record.md` needs to move to `support-log.md`.
