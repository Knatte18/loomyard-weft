---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/06-round-gate-headroom.md (Intent, loomCLI.wire paragraph)
    summary: Card 6 reads the origin record eagerly in wire through fabricengine.ReadOrigin, which mkdirs the weft lock dir and seeds the weft exclude, turning a read error into a wire error and breaking every hubLocation-based wire test the card does not list
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/03-lint-pattern-verb.md (TestLintPatternCmd_OverARepository); _lyx/plan/00-overview.md (new-tests-where-the-record-counted-none)
    summary: Card 3 creates a new top-level integration test for lint-pattern although the record's PATTERN-test-economy constraint says the verb's tests extend the test that covers lint-comments's envelope, which exists
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/10-pattern-overview-trim.md (Intent and target list)
    summary: Card 10 appends detail to existing pattern/ background files but names none of them as targets, so its write surface is not declared
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/10-pattern-overview-trim.md (Create list and Intent)
    summary: Card 10 says its five new background files are for the longest lines without one, but PATTERN-documentation-lifecycle at 242 characters is the third longest such line and has no file planned, so trimming it would break D4
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/01-gofmt-guard.md (Create list and Intent); _lyx/plan/00-overview.md (new-tests-where-the-record-counted-none)
    summary: Card 1 plans two top-level tests and a raise of two on the cmd/lyx row, departing from D7's raise of one on the premise that the in-memory cases cannot share the guard function, which does not hold since the guard can host them as subtests
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/06-round-gate-headroom.md (Edit list, roundGateFixture.runGate)
    summary: Card 6 changes roundGateFixture.runGate's parameters (it gains an optional merge base reader) but its member glyph carries no re-sign arrow
  - id: F7
    severity: LOW
    class: design
    location: _lyx/plan/03-lint-pattern-verb.md (readOverview)
    summary: In the working-tree mode readOverview compares os.ReadFile bytes with a git show blob, so on a checkout with core.autocrlf the head counts CRLF bytes the base lacks and reports growth that is not there
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/plan/06-round-gate-headroom.md (ImpactSummary)
    summary: The ImpactSummary says NewVerifyGate has four callers, but there are five call sites, all on the card, namely TestVerifyGate_Scenario, runGate, wholeDiffGate, resolvePlainGateClosure and TestGateParity_VerifyGate
---

### [BLOCKING] Card 6 moves the origin read into wire, which writes and fails there

**Location:** `_lyx/plan/06-round-gate-headroom.md`, Intent: "`loomCLI.wire` tells `env.VerifyMergeBase` the `pairMergeBase` reader for every pair run: it reads the fabric origin record once at wire time through `fabricengine.ReadOrigin` … a read error at wire time is wire's error."

**Issue:** `fabricengine.ReadOrigin` (`internal/fabricengine/origin.go:82`) is not a plain read.
It calls `originLockPath`, which calls `ensureWeftLockDirAt` (`internal/fabricengine/recordsgit.go:59`).
That function runs `os.MkdirAll` on the weft lock dir and then `seedWeftArtifactExcludes`.
The exclude seeding calls `mutateGitExclude`, which resolves the weft repo's common dir through `gitrepo.New(dir).CommonDir()` and fails when the weft sibling is not a git repository.
Three consequences:

1. Every loom wire now writes into the weft worktree and may fail before any row builds.
   `wire` is otherwise built to open nothing at wire time; see the comment at `internal/loomcli/wiring.go:525` and the lazy `ParentBranch` closure just above it.
2. The decision record says the reader is built "as darn's wiring builds its merge-base reader" and that "a merge-base read error is a returned gate error, as it is for darn".
   Darn's reader (`darnMergeBase`, `internal/loomcli/darnwiring.go:46`) reads the parent on each call.
   Card 6 turns an origin read error into a wire refusal for every loom verb that wires, a new failure mode the record does not license.
   It has no way forward and no refusal-spec row, contrary to `PATTERN-refusal-way-forward`.
3. The Tier 1 wire tests build their hub by hand with `hubLocation` (`internal/loomcli/wiring_test.go:130`), which creates no weft repository.
   Under card 6, `ReadOrigin` errors in each of them, so `wire()` fails.
   That covers the tests in `wiring_test.go`, `wiring_seams_test.go`, `wiring_darn_test.go` and `wiring_parentbranch_integration_test.go`.
   The card lists only `TestWire_DarnRecipe`.
   Even that test's planned assertions cannot hold.
   The loom-seed step "asserts the reader is set" and the darn-seed step "asserts both are set", but the fixture writes no origin record, and the card sets the reader only when one is found.

**Fix:** Keep the reader lazy and decide at wire time without a mutating read.
For example, wire can stat `fabricengine.OriginRecordPath(location)` and set `pairMergeBase` only when the record file exists, leaving the read and its errors to the gate as darn has them.
Alternatively, it can use a read that creates nothing, in the manner of `ReadOriginFor`.
Then name the fixture work: `TestWire_DarnRecipe`'s steps that assert the reader is set must write an origin record into the fixture, and the card's Edit list must cover any other wire test whose assertions change.

### [MEDIUM] Card 3 adds its own test where the record says to extend the comment lint's

**Location:** `_lyx/plan/03-lint-pattern-verb.md`, `plan:internal/loomcli#TestLintPatternCmd_OverARepository`; Shared Decision `new-tests-where-the-record-counted-none`.

**Issue:** The decision record's constraint reads: "`PATTERN-test-economy`: … the verb's tests extend whatever test covers `lint-comments`'s envelope where one exists."
One exists: `TestLintCommentsCmd_OverARepository` in `internal/loomcli/lintcomments_integration_test.go` drives the verb over one repository through the dirty-tree, clean-tree and `--base` steps.
Card 3's new test asserts the same envelope shape over the same three steps.
The plan's reason, that "the fixture and name are the comment verb's", does not justify the new test.
The fixture is a scratch repository that can also hold a `PATTERN.md`, and renaming a test is an edit.
The plan overrides a settled constraint in a Shared Decision instead of carrying it.

**Fix:** Extend `TestLintCommentsCmd_OverARepository` into one scenario over both lint verbs, renamed to say so.
In each existing step, also write or commit a `PATTERN.md` past `pattern.HeadroomBytes` and assert `lintPatternCmd`'s envelope.
Drop the new top-level test and the `internal/loomcli` budget raise, and amend the Shared Decision to cover card 1 only.

### [MEDIUM] Card 10 writes existing background files it does not target

**Location:** `_lyx/plan/10-pattern-overview-trim.md`, Intent: "an entry that has one gets the detail appended there".

**Issue:** The card's targets are `PATTERN.md` under Prosa and five new files under Create.
The Intent appends trimmed detail to the existing `pattern/PATTERN-*.md` background files of the entries it shortens, and with 31 entry lines over 230 characters, most of those entries already have one.
None of those files is a target, so the card's write surface is undeclared.
The plan-level reviewer and the implementer cannot see which files change.
Dependency derivation also cannot see the overlap with cards 4, 6 and 7, which edit `pattern/PATTERN-gate-self-check-parity.md` and `pattern/PATTERN-test-speed.md`.

**Fix:** Name the entries the card trims, and list each existing background file it appends to in the Prosa group beside `PATTERN.md`.

### [MEDIUM] Card 10's choice of new background files skips PATTERN-documentation-lifecycle

**Location:** `_lyx/plan/10-pattern-overview-trim.md`, Create list and Intent ("the five entries named under Create, whose lines are the longest without a background file").

**Issue:** Measured on the current `PATTERN.md`, the entry lines with no background link, longest first, are:

- `push-both-sides` (267)
- `edit-directive` (251)
- `documentation-lifecycle` (242)
- `wait-mark-display-only` (241)
- `spawn-color` (234)
- `hub-wide-config` (230)

The card picks `hub-wide-config` and skips `documentation-lifecycle`, so its own selection rule is false.
That leaves the entry in a contradiction.
The Intent says the cut comes mostly from lines over about 230 characters.
If the entry is trimmed, D4 requires its detail to move into a background file "created when the entry has none", and none is planned.
If it is left alone, the card should say so.

**Fix:** Either add `pattern/PATTERN-documentation-lifecycle.md` to Create, or state that the entry is left untrimmed because its detail lives in `docs/overview.md#documentation-lifecycle`, and correct the selection sentence.

### [LOW] Card 1's second top-level test rests on a false premise

**Location:** `_lyx/plan/01-gofmt-guard.md`, `TestGofmtFinding` and the budget sentence; Shared Decision `new-tests-where-the-record-counted-none`.

**Issue:** D7 says the `cmd/lyx` row "rises by one in the same commit", which is one new test.
The plan raises it by two because "the guard's in-memory cases cannot share the function that carries `//lyx:guard` and does the real scan".
Nothing prevents a guard test from running in-memory subtests before its scan.
The split in `testbudget_test.go` is a precedent, not a constraint.
The departure is small, but it is a decision the record settled, and the plan should not reverse it on a false premise.

**Fix:** Fold the in-memory cases into `TestGofmt_ModuleFilesAreFormatted` as table subtests and raise the row by one, as D7 says.
Alternatively, keep the split and correct the stated reason to the precedent the plan follows.

### [LOW] runGate's re-signed glyph has no arrow

**Location:** `_lyx/plan/06-round-gate-headroom.md`, Edit `internal/loomshed#roundGateFixture.runGate`.

**Issue:** The Intent says "`roundGateFixture.runGate` takes an optional merge base reader", which changes its parameters.
Its current signature is `func (f roundGateFixture) runGate(t *testing.T) (passed bool, findings string)` (`internal/loomshed/verifygate_integration_test.go:134`), and the glyph carries no re-sign arrow.

**Fix:** Write the arrow, for example `internal/loomshed#roundGateFixture.runGate -> func (f roundGateFixture) runGate(t *testing.T, mergeBase func() (string, error)) (passed bool, findings string)`.

### [LOW] The working-tree mode counts CRLF bytes against an LF blob

**Location:** `_lyx/plan/03-lint-pattern-verb.md`, `readOverview` paragraph.

**Issue:** With an empty base, `Lint` compares HEAD's `PATTERN.md`, read through `git show` as the LF blob, with the working-tree file read by `os.ReadFile`.
`.gitattributes` pins no `eol` for `PATTERN.md`.
On a checkout with `core.autocrlf=true`, which the Windows tooling under `sandbox/win` implies is a supported host, the working tree holds a CR per line.
An untouched overview then counts as growth of about one byte per line, and once the LF size sits within that margin of the line, every card gate fails with no edit made.
`commentlint` compares lines, so it never had this exposure.

**Fix:** Normalize CRLF to LF on both sides in `readOverview` before handing the contents to `pattern.Headroom`, and add a row or step that pins it.
Alternatively, pin `PATTERN.md text eol=lf` in `.gitattributes` on this card.

### [NIT] NewVerifyGate caller count

**Location:** `_lyx/plan/06-round-gate-headroom.md`, ImpactSummary.

**Issue:** "whose four callers are on this card" does not match the code.
There are five call sites:

- `internal/shedrecipe/entries_gate.go:212`
- `internal/loomshed/verifygate_integration_test.go:50`, `:136` and `:145`
- `internal/webstercli/verify_parity_integration_test.go:83`

All five are covered by the card's glyphs.

**Fix:** Say "whose call sites are all on this card" instead of pinning a count.

## Focus verdicts

- Fidelity:
  - D1 to D8 and every Constraint are carried.
    The refusal-spec row and the parity background-file pair are on card 6, the latter also on card 4.
    The gate-per-tier table is on cards 4 and 6, the implementer stencil and `wordingClaims` on card 4, and the doc.go and `docs/overview.md` updates on cards 2, 3, 4, 6 and 8.
  - `loom-rubric-webster-review.md` mentions `lint-comments` only for comment line breaks, so not naming the new lint is correct.
  - The `ly:cli` index is generated from the cobra tree, so no file needs an edit.
  - The departures from the record are F2 and F5.
- Card order and Uses: no card depends on a later one.
  Card 3 uses card 2's `Headroom`, card 4 names card 3's `patternlint.Lint`, and card 6 uses cards 3 and 5.
  Card 9 uses card 8, and card 10 comes last.
- Verify coverage: `## verify:` follows the stencil's order and runs the integration tier, which covers `lint-pattern`, the round gate, `gatecli` and `shedverbs`.
  The `tmux` (`pairteardown`) and `llm` (`orchcli`) files are compiled by `go vet`, which the rubric exempts.
  This is not a defect.
- Test economy: card 8's probe test covers exports no existing test pins, and card 7 adds no test.
  Cards 1 and 3 are F5 and F2.
  Every card that adds a top-level test lists `cmd/lyx/testdata/test-budget.yaml`.
- Glyphs:
  - `NewVerifyGate` carries its arrow.
  - `shedrecipe.Env` gains a field, which takes no arrow.
  - Card 5's rename is a `Rename` card.
  - The one missing arrow is F6.
- Attack surface: card 6 states the headroom lint's bound, the standalone skip and the darn-only `wholeDiff` flag with its nil-reader error.
  This is not a defect beyond the wire-time failure mode in F1.
- Card 10: it keeps every name, trigger, requirement and `(test)` mark, counts the link bytes, and licenses new background files through D4.
  F3 and F4 are the gaps.
