---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 (covered-package file record) and D5 (non-Go file miss)
    summary: A non-Go file in a covered package's subdirectory, such as an embedded stencil under contracts/stencils/loom/, is neither recorded nor compared, so its change narrows importers to nothing selected
    fix: In D2, record per covered package every tracked file that impactset's path-to-package rule maps to that package, meaning every file under its directory outside a nested package directory, go:embed targets included, not only the files directly in the directory. In D5's last bullet, compare that same set. Add a D5 acceptance criterion where an edit to an embedded file in a subdirectory of covered package A runs importing package B whole with a miss reason.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D2 (covered set), D4 and D5 (compared packages)
    summary: A linked in-module package with no covered block, such as a package of only constants or types like internal/lymdirs, which declares no function, is never compared, so changing its constant or type selects nothing
    fix: In D2, record the file set and hashes of every in-module package the test binary links (the coverpkg list), not only the packages a test covered a block of. In D4 and D5, compare every package so recorded, so a non-body change in a linked package with no covered block is a D5 miss. Reword the D2 acceptance criterion so it no longer says not stale when only an uncovered package changed, and state the staleness rule for linked-but-uncovered packages explicitly. The cost question for that staleness goes with F4's parent question.
  - id: F3
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D5 (non-body change miss) and Open risks (third bullet)
    summary: The non-body miss, judged from the map's own Commit, fires on most landings to widely linked packages, so in common flows importing packages fall back to whole runs; the record leaves that hit rate unmeasured in Open risks
    fix: Measure, from main's history over a recent window, how often a landing changes a widely linked package outside function bodies, and record the number with its source in Constraints. Then narrow D5's non-body rule so an added declaration that no unchanged code can reach is no miss, the way a plain function already is. That covers an added const, an added var without an initializer that runs code, and an added type with no methods. Keep as misses a changed or removed declaration, a struct field change, a var initializer and a method-set change. Add a D5 acceptance criterion for an added const in A being no miss. If the measured rate still leaves coverage close to diff once testmap's cadence is counted, name that dependency as a question for the parent in Open risks, tied to note testmap-upkeep, in place of the current third Open risks bullet.
  - id: F4
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md Open risks (first bullet) and support-log.md Q10
    summary: Open question for the parent - may D2 change testmap's staleness rule and raise its rebuild cost now, when note testmap-upkeep says it owns the whole staleness policy, dependency drift included
    fix: Raise to the run's parent. Once the parent's answer is recorded with lym loom decision add, the writer either keeps D2's staleness change and drops the Open risks line, or removes D2's staleness rule and states in D5 that a dependency-drift miss lasts until testmap rebuilds the map by its present rules.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4 (body text comparison) against D5 (outside function bodies)
    summary: A change to a function's signature or receiver with an identical body is neither a changed function under D4 nor plainly a non-body change under D5, so it can select nothing
    fix: In D4, define a changed function as one whose whole declaration text differs, receiver, signature and body together, ignoring comments and formatting. Add a D4 acceptance criterion where changing a method's receiver from value to pointer with the same body selects the B tests that covered it.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5 (fifth bullet, other than comments)
    summary: Compiler directives are comments, so a changed go:embed, go:build or go:linkname line counts as no change and selects nothing
    fix: In D5's fifth bullet, state that a comment exception never covers a compiler directive line, one opening with //go followed by a colon, so an added, removed or changed directive line is a miss. Add it to the D5 acceptance criterion that lists the miss and no-miss edits.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D3 (union) and D6 (selected invocation)
    summary: The D3 union is not limited to the tier for guard tests and does not drop the tests the watch skip names, so a selected invocation can run zero tests and trip the existing empty-tier check, a false gate failure
    fix: In D3, state that the union holds only tests of the tier, guard tests included, and drops the package's tests that the diff's watch skip names. State that a package whose union is then empty counts as none selected and is left out of the selected invocation. Add a D3 acceptance criterion where an importing package whose only union member is an unwatched watch-marked guard test runs no test, and the gate passes.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4 (go/parser parse) and D5 (miss list)
    summary: Neither D4 nor D5 says what a covered file that does not parse with go/parser, at the map's Commit or in the working tree, does to the scope
    fix: Add a D5 bullet saying a changed covered file that does not parse on either side is a miss for the importing package, with the reason in the scope report.
  - id: F9
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D4 and D5 (working-tree hash against the map's blob hash)
    summary: Map hashes come from blobs at Commit while D4 and D5 hash working-tree bytes, so on a checkout whose core.autocrlf converts Go files every file reads as changed, and nothing states that bound
    fix: In D4, state that the working-tree side is hashed as git would store it, through the checkout's clean filter, or add to D5's Bound that a checkout converting line endings misses for every importing package.
  - id: F10
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 (A map's recorded hashes are therefore the content at its Commit)
    summary: testmap reads the prime's dirty files once before its walk and keys blocks from the working tree after the tests run, so an edit made in a covered package mid-walk and reverted later enters the map under Commit's hashes
    fix: In D2, state that testmap repeats its dirty check over every covered package's directory after a package's tests end, and discards that package's map when one is dirty, as it does for files changed at HEAD.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md Constraints (PATTERN-gate-slot bullet)
    summary: The constraint says the scope reads maps inside the gate's own slot, but computeScope runs before any tier takes its slots
    fix: In the PATTERN-gate-slot constraint, replace "the scope reads maps inside the gate's own slot" with "the scope reads maps before any tier takes its slots, and starts no build".
---

### [BLOCKING] Embedded non-Go files in a covered package's subdirectory are never recorded

**Location:** `decision-record.md` D2 and D5's last bullet.

**Issue:** D2 records "every file its directory tracks" for a covered package, and testmap lists a directory through `fabricengine.Fabric.CodeDirAt`, which returns only direct files (`internal/fabricengine/codereads.go:12-21`).
`contracts/stencils` embeds files from `loom/`, `landing/` and other subdirectories that hold no Go file.
impactset maps such a file to `contracts/stencils` (`internal/impactset/derive.go:99-117`), so the package counts as changed and its importers enter the narrowed set.
No Go function changed, so D4 selects nothing, and D5's non-Go rule never saw the file, so no miss fires.
The importers' tests that render the stencil do not run, which breaks the Goal's "any doubt widens".
The walkthrough lens found the same defect independently.

### [BLOCKING] Linked packages with no covered block are never compared

**Location:** `decision-record.md` D2, D4 and D5.

**Issue:** Go's cover tool counts only statement blocks inside function bodies.
A package that B's tests reach only through constants, types or var initializers has no covered block, so under D2's wording it is not a covered package and nothing compares it.
`internal/lymdirs` declares no function at all (`dirs.go` and `doc.go` hold zero `func` lines), and it is linked by most of the module.
A const change there puts B in the `diff` set, but B reports `none selected`.
The D2 criterion "not stale when only an uncovered package changed" bakes that gap in.

### [BLOCKING] The non-body miss leaves importing packages whole in common flows

**Location:** `decision-record.md` D5's fifth bullet and the third Open risks bullet.

**Issue:** Focus item 1 asks whether a worktree whose base predates a landing falls back to whole runs in common flows.
It does.
D4 and D5 compare the map's own `Commit` with the working tree.
Any change outside function bodies to any covered package between them is a miss, whichever side it lies on.
Such changes include an added const, type or struct field, a new method, or an import of an uncovered package.
I measured `main` from 2026-11-04, with a rough pattern over `git show -U0` for `type`, `const`, `var`, method headers, import lines and struct fields.
102 of the 117 commits that touch non-test Go under `internal/` or `cmd/` carry such a change.
For widely linked packages the counts are:

- `internal/fabricengine`: 17 of 26 commits
- `internal/configreg`: 7 of 12
- `internal/lymdirs`: 6 of 9
- `internal/configengine`: 3 of 6

So several such landings reach main each day.
A map stays usable only until the next one, and testmap rebuilds 5 maps per 5 landings across about 170 packages.
In steady state nearly every importing package that links one of these packages misses, and `coverage` runs close to `diff`.
F2's fix widens the compared set and makes this worse.
The third Open risks bullet names this effect, but leaves its size, a fact the design's whole benefit rests on, for later.
The rubric's deferred-research bullet flags exactly that.
An added declaration that only changed code can reference widens for the same reason D5 already rejects for a new plain function.

### [MEDIUM] Parent question: D2's staleness change ahead of note `testmap-upkeep`

**Location:** first Open risks bullet; support log Q10.

**Issue:** D2 adds covered-package drift to testmap's staleness rule and raises each build's cost.
Note `testmap-upkeep` says it "owns the whole policy", dependency drift included.
No `Decision for gate-scope-by-coverage-map:` board note and no `Added after Discussion` entry answers this.
It is a design overlap with another board entry, which only the parent settles.

### [MEDIUM] A signature or receiver change neither selects nor misses

**Location:** D4's changed-function rule against D5's "outside function bodies".

**Issue:** D4 compares body text only.
Block keys name their function by the whole `FuncDecl` span, signature included (`internal/testcover/blockkey.go:150-156`), so a receiver changed from value to pointer with an identical body falls between D4 and D5.
That edit changes what callers see, yet it selects no test and raises no miss.

### [MEDIUM] Changed `//go:` directives count as comments

**Location:** D5's fifth bullet.

**Issue:** D5 exempts comments from the non-body miss.
A retargeted `//go:embed`, or a `//go:build` line that moves a file in or out of the build, is a comment to `go/parser`.
So it is neither a body change nor a miss, and the tests that read the embedded value do not run.

### [MEDIUM] The selected invocation can run zero tests and fail the empty-tier check

**Location:** D3's union and D6's selected invocation.

**Issue:** D3 adds "every `//lym:guard` test of the package" with no tier limit, and does not drop the tests the watch skip names.
The existing `emptyTierFailure` in `internal/gatecli/tierrun.go` exempts only a call whose own words after `--` hold a filter, not the run's args.
So a selected invocation whose package runs nothing can fail the gate.
That happens when the package's only member is a guard of another tier, or an unwatched guard that the `-skip` removes.
The interactions lens raised the same watch-skip gap as an inconsistency with D1's "watch skips act as under `diff`".
This finding carries that gap too.

### [LOW] An unparsable covered file has no stated outcome

**Location:** D4 and the D5 miss list.

**Issue:** D4 parses both sides with `go/parser` and names no outcome for a file that fails to parse.
The plan writer could make that a refusal or a silent skip, and only a miss keeps the Goal's widen-never-narrow rule.

### [LOW] Line-ending conversion turns every file into a change

**Location:** D4 and D5's hash comparison.

**Issue:** Map hashes are blob hashes through `fabricengine.Fabric.CodeFileAt` (`internal/gatecli/testmap.go:436-441`), while the scope hashes working-tree bytes.
`.gitattributes` pins LF only for listed files.
So a checkout whose `core.autocrlf` converts Go files misses on every package.
That is safe but defeats the decision, and no bound says so.

### [LOW] testmap's dirty check runs once before its walk

**Location:** D2's claim that recorded hashes are the content at `Commit`.

**Issue:** `prepare` reads `Status` once (`internal/gatecli/testmap.go:270-277`).
`changedFiles` then compares HEAD with HEAD, and `KeyBlocks` reads the working tree.
An edit to a covered package in the prime during the walk is built into coverage that the map records under `Commit`'s hashes.
D2 widens that window from one directory to every covered package.

### [LOW] The gate-slot constraint misstates when maps are read

**Location:** the PATTERN-gate-slot bullet under Constraints.

**Issue:** `computeScope` runs at `internal/gatecli/run.go:82`, before the per-tier `AcquireSlots` at `internal/gatecli/run.go:686`.
So D8's map read happens outside every slot, which PATTERN-gate-slot allows, since the read starts no build or test.
The constraint's wording is wrong, not the design.

## Rejected

- Interactions lens F2 (the watch skip is not carried into the selected invocation) is merged into F7, which covers the same gap and its gate failure.
- Interactions lens F3 is kept as F11 at LOW, not MEDIUM: the misstatement changes no behaviour, since reading maps outside a slot breaks no rule.
- Walkthrough lens finding 1 duplicates bound-breaker finding 1 and is merged into F1.
