---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:117-118,135-136 (D7, D9; Scope #325 §5)
    summary: D9 forbids deleting any integration live test or any of its assertions, yet D7 drops the CLI remedy-string assertion in integration-tagged fabriccli/cli_test.go and #325 §5 deletes one of two integration conflict-envelope tests; neither removal states its bound or its exception to D9
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:70,74 (D3)
    summary: D3's spawn-import rule bans only os/exec and gitexec, so a kit importing gitkit or hubforge spawns git unflagged; kits' non-test files sit outside Tier Purity, which scans only _test.go files
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:70-78 (D3)
    summary: D3 does not say which Testkit rules are scan-enforced and which are review discipline; the lyxbin go-build-only bound and the starts-no-process and asserts-nothing rules are held by nothing named
  - id: F4
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:92,94,249 (D4, Notes)
    summary: D4 moves internal/websterengine/config_test.go to an integration-tagged file as an untagged gitkit caller, but that file is deliberately spawn-free and only names gitkit.SeedConfig in a comment; D4's Bound also omits which tests leave the default go test run
  - id: F5
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:227-243 (Acceptance criteria)
    summary: several Scope items have no checkable criterion - shedfake Call helpers, #324 §6 log capture, envelope, bouncer fixture, buildLyxBinary and slices replacements, #325 §6 reap-cycle table and lyxcwd.Location builder
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md:236
    summary: the Tier Purity / Hermetic Env sections criterion and the fabricengine fabricFixture criterion are run together on one line, so they render as one bullet
  - id: F7
    severity: LOW
    location: _lyx/discussion/decision-record.md:176-191 (D13)
    summary: D13's Kept-unchanged reasons and Notes for test-suite-derived-guards are audit narrative and part-2 notes, not decisions the plan writer acts on; they belong in support-log.md once their constraints are confirmed elsewhere in the record
  - id: F8
    severity: LOW
    location: _lyx/discussion/decision-record.md:88,97-98,106,211
    summary: D4's narrative of the two scans' current disagreement and the survey tallies in D4/D5 rationale and Auto-mode assumptions are perishable Plan-time facts the writer's stencil says not to gather
  - id: F9
    severity: NIT
    location: _lyx/discussion/decision-record.md:89,95 (D4)
    summary: D4 states every gitkit export but HermeticGitEnv spawns git, but gitkit.RepoFixture is an existing non-spawning export; the Bound speaks only of future ones
---

### [MEDIUM] D7 and #325 §5 removals conflict with D9's protected list

**Location:** `_lyx/discussion/decision-record.md` D7 (lines 117–118), D9 (lines 135–136), Scope's #325 §5 bullet (line 25).

**Issue:** D9 protects "every `smoke`/`integration` live test" and says these are "never deleted, and every assertion they make survives".
D7 says each remedy string "keeps exactly one test, in the engine", so CLI substring assertions go.
One such CLI assertion lives in an integration-tagged file: `internal/fabriccli/cli_test.go` is `//go:build integration` and asserts `"has no recorded warp binding"` at line 631.
#325 §5 (pulled into Scope wholesale) keeps "the unit test and one integration test" of the fabriccli conflict envelope, deleting either `envelopecontract_integration_test.go`'s `TestRunCLI_MergeConflictEnvelopeKeepsPartialFalse` or the first half of `merge_cli_integration_test.go`'s `TestRunCLI_MergeInConflictThenContinueConcludes`, both integration tests.
So the plan writer gets two contradictory rules.
Neither D7 nor Scope's §5 bullet states what its removal can let through or what bounds it.
D8's replacement of nil-check tests is not affected: the nil-check tests sit in untagged `wiring_test.go`/`wire_test.go`, and D8 keeps value and identity tests.

**Fix:** In D9, state the exception: a D7 remedy-string assertion or a #325 §5 duplicate may be removed from a protected test only when the same behaviour stays asserted elsewhere, and the removal is itemised in the test-count accounting.
In D7, add a Bound: dropping CLI substring assertions lets a CLI-side rewrite of the message through only if the generic pass-through assertion misses it, and that assertion is what bounds it.

### [MEDIUM] D3's spawn-import rule misses `gitkit` and `hubforge`

**Location:** `_lyx/discussion/decision-record.md` D3 (lines 70 and 74).

**Issue:** "A kit spawns nothing: it imports neither `os/exec` nor `internal/gitexec`".
`gitkit` (`MustRun`, `SeedConfig`, `CopyRepo`, `GitStatusPorcelain`) and `hubforge` (`NewHub`) exist to spawn git, and a kit can import them without importing `os/exec` itself.
Tier Purity does not catch it either: `cmd/lyx/tierpurity_test.go` walks only `*_test.go` files, and a kit's code is non-test `.go`.
An untagged test calling such a kit spawns git in Tier 1 with no scan flagging it, exactly what D3's rationale ("kits that spawn nothing are safe in Tier 1 tests") relies on not happening.

**Fix:** Extend D3's spawn-import rule and its scan to ban `internal/gitkit` and `internal/hubforge` imports from kits, and add a planted `gitkit` import to the acceptance criterion's plants.

### [MEDIUM] D3 does not split scan-enforced rules from review discipline

**Location:** `_lyx/discussion/decision-record.md` D3 (lines 70–78).

**Issue:** The scan covers the importer, `*cli` and spawn-import rules.
"Starts no process, tmux server or agent" beyond those imports and "asserts nothing ... beyond `t.Fatalf`" are not scanned, and D3 does not say they are review discipline, the way D6 states it for its clauses.
The `lyxbin` Bound says the exemption "lets one kit spawn `go build`, and nothing else", but the scan exempts `lyxbin` from the spawn-import rule by name, so nothing mechanical keeps it to `go build` of `./cmd/lyx`.
The `lyxbin.` Tier Purity token bounds where it can be called, not what it spawns.

**Fix:** In D3, state per rule how it is enforced: the importer, `*cli` and spawn-import rules by the scan; the no-process, asserts-nothing and `lyxbin`'s go-build-only limit as review discipline, stated as such in the `CONSTRAINTS.md` section.

### [MEDIUM] D4 tags a deliberately spawn-free test and leaves its Tier 1 loss unstated

**Location:** `_lyx/discussion/decision-record.md` D4 (lines 92 and 94), Notes for the plan writer (line 249).

**Issue:** D4 moves "existing untagged `gitkit` calls (in `internal/fabricengine/export_test.go`, `clone_test.go`, `internal/websterengine/config_test.go` ...)" to integration-tagged files, and the Notes say all three "call gitkit helpers".
`internal/websterengine/config_test.go` calls no gitkit helper: its file comment says it "stays untagged and spawn-free (Test Tier Purity Invariant)", and its only `gitkit.` text is a doc comment calling `seedConfig` "a plain-filesystem stand-in for gitkit.SeedConfig".
Under D4's raw-substring rule that comment trips the scan; the right remedy is rewording it (as Open risks already says for comments), not tagging the file, which would take its config tests out of the default run for nothing.
D4's Bound lists the helpers untagged tests lose, but not the coverage cost: the fabricengine tests that call `gitkit.MustRun` today run only under `-tags integration` afterwards.

**Fix:** Drop `config_test.go` from D4's list and from the Notes bullet, and say a file that only names a gitkit helper in a comment rewords it and stays untagged.
Add to D4's Bound that the tests using the moved helpers leave the default `go test ./...` run and run under `-tags integration`.

### [MEDIUM] Scope items with no acceptance criterion

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria (lines 227–243) against Scope (lines 14–28).

**Issue:** Walking each Scope bullet, these have no criterion that makes them checkable:

- #324 §2's producer `Call` helpers in `shedfake` (D9 only fixes their semantics).
- #324 §6: log capture, JSON envelope decoding, the shedadapters bouncer fixture, `buildLyxBinary` and the `containsPath`/`equalStringSlices` replacement by `slices`.
  The first criterion requires the kits to exist, not that the per-package copies are gone.
- #325 §6's reedcli reap-cycle table and the shared `*lyxcwd.Location` builder.

The other bullets map to a criterion.

**Fix:** Add one criterion for the #324 §6 items (no package keeps its own log-capture, envelope-decoding or `buildLyxBinary` copy; `containsPath`/`equalStringSlices` are gone; shedadapters uses one bouncer fixture), extend the `shedfake` criterion to the `Call` helpers, and add one for the reap-cycle table and the single `*lyxcwd.Location` builder.

### [LOW] Two acceptance criteria run together on one line

**Location:** `_lyx/discussion/decision-record.md:236`.

**Issue:** "... carry the D6 clauses.- fabricengine tests have no `fabricFixture` ..." is one line, so markdown renders the fabricengine criterion inside the `CONSTRAINTS.md` one.

**Fix:** Break the line so each criterion is its own bullet.

### [LOW] D13's kept-unchanged reasons and part-2 notes belong in the support log

**Location:** `_lyx/discussion/decision-record.md` D13 (lines 176–191).

**Issue:** Completeness first: the constraints in "Kept unchanged" already live elsewhere in the record (the Fabric Vocabulary and seam-scan rules in `## Constraints`, the Completion Signal rows in the Runner-fixture criterion), and "Changed by this task" plus "no invariant's intent is weakened" tell the plan writer not to touch anything else.
What remains is audit deliberation (why each invariant stays) and notes addressed to `test-suite-derived-guards`, which this task's plan writer does not act on; the glyph-plan bullet repeats `## Scope`.
`Plan-Write` loses nothing by their move.

**Fix:** Keep D13's Decision, its "Changed by this task" list and its rationale; move the "Kept unchanged" reasons and the part-2 notes to `support-log.md` (the interview's audit item).

### [LOW] Perishable survey narrative and tallies in the record

**Location:** `_lyx/discussion/decision-record.md` lines 88, 97–98, 106 and 211.

**Issue:** D4's line 88 narrates which token list holds which helper today; the plan writer needs the restated rule (lines 89–91) and the Bound, and the history is already in interview item 7.
The tallies (84 private helpers, ten exports and twenty edits, ~190 sites, 10/36/82 in Auto-mode assumptions) are snapshots the Notes already tell the plan to recompute, and the stencil puts that class of fact at Plan time.

**Fix:** Move line 88 to `support-log.md`'s interview item 7 (it is already there in substance) and drop it from D4; state D4/D5's rationale and the drift assumption without counts.

### [NIT] An existing non-spawning gitkit export

**Location:** `_lyx/discussion/decision-record.md` D4 (lines 89 and 95).

**Issue:** `gitkit.RepoFixture` is an exported type that spawns nothing, so "every `gitkit` export spawns git except `gitkit.HermeticGitEnv`" is inexact today, not only for future exports.
It is harmless because only `CopyRepo`, itself banned, produces one.

**Fix:** Name `RepoFixture` in the Bound as banned along with `CopyRepo`, which is its only producer.

## Verdicts on focus items with no finding

- D1's cycle-forced local copies are a closed, named set (`shuttleengine`'s own tests, `shedrecipe`'s and `landingshed`'s own builders), each "exactly one local copy", and the consumer criterion excepts only "the cycle-forced cases named in D1": bounded, not a defect.
- Reverse completeness: interview item 1's "`pattern-init` later moves `CONSTRAINTS.md`" is board context, not a constraint; the record already says the invariant lands in `CONSTRAINTS.md`.
  No other requirement lives only in `support-log.md`.
