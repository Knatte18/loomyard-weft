# Decision record — lighter-test-rules

## Goal

A change to a stencil's prose costs no test change.
Tests keep pinning only what Go fills, reads or names (D1), and review guards prompt wording instead.

## Scope

In:

- Delete `contracts/stencils/claims_test.go` whole: the `wordingClaims` table, its helpers, `TestStencilClaims`, `TestBranchOf` and `TestSectionOf`, the provenance-citing protected claims and the `loom-discussion-spec.md` rows included.
- Move the claims_test rows D1 keeps into kept tests in `contracts/stencils` (D2).
- Prune every other test assertion on stencil or rendered-prompt text by D1, wherever it sits (`*template*_test.go`, `*prompt*_test.go` and any other test); the plan writer sorts each assertion by D1 rather than by file.
- Change the rules that ask for a test per stencil change, and make review guard prompt wording (D3, D4).
- Fix the doc comments of kept test files that point at `claims_test.go` or describe the dropped prose assertions.

Out:

- Assertions on text built in Go source (a refusal message, a deny notice, a Go-built reprompt such as `shuttleengine`'s gate reprompt): that text is Go behavior, and `PATTERN-refusal-way-forward` asks for its test.
- Spec prose beyond the `loom-discussion-spec.md` rows that leave with `claims_test.go`.
- The doc rule (`doc.go`, `docs/overview.md` and `PATTERN.md` in the same commit).
- Any stencil edit beyond the plan stencil and rubric text D3 and D4 name.
- `CLAUDE.md`, which carries no per-stencil-change test rule.
- A mechanical scan that refuses a new prose assertion: review enforces the rule, as it does the rest of `PATTERN-test-economy`.

## Decisions

### D1 — The keep-or-drop rule for a stencil assertion

Decision: an assertion about stencil or rendered-prompt text stays when the text it checks comes from Go or is read by Go, or when it enforces a rule across a stencil family; otherwise it goes.
It stays when it checks:

- a marker set, a required-marker error, a Fill/FillOptional round-trip, or the order of two markers Go fills;
- a value Go filled into a marker, or a fragment or branch Go chose by its inputs, anchored on the shortest phrase that tells the branches apart;
- a stencil naming a Go-declared value: a constant (`shuttleengine.MessageTail`, `shedrun.ParkMarkerFileName`, `gatecli.SlotBusyExit`), or a key or field name the test reads from the Go definition that parses it (a constant or a struct tag), such as the webster master's digest and outcome-key lists and the fork report schema;
- a cross-stencil rule: a PATTERN scan over stencils, the registry, the rubric marker allowlist, the citation scan, and the twill host-neutral ban (`hostNeutralClaims`: a twill stencil names no host value, since another host renders it).

It goes when it checks static prose of one stencil: a `must` phrase every render input produces alike, a `mustNot` phrase banned from one stencil (`droppedConceptClaims`, the orch scribe-handoff ban), and a literal key name with no Go definition the test can reference, whatever `why` or provenance it cites.
A test function left with no assertion is deleted; one left with kept assertions stays with only those.
An unused helper left behind is deleted.

Rationale: the operator decision of 2026-10-15 sorts stencil tests into markers and Fill round-trips, cross-stencil scans and registry, and wording claims; this rule sorts a single assertion, which is the board's open item on prose assertions that are marker or structure checks.
A drift in Go-filled, Go-read or Go-named text breaks a run; a drift in prose changes how an agent behaves, which a substring test cannot judge.

### D2 — Where the kept claims_test rows go

Decision: from `claims_test.go`:

- rows naming a Go-declared value (D1's third keep) become one table-driven test in `contracts/stencils`, one row per stencil and value, each row naming a registered stencil and failing when the stencil no longer names the value;
- marker-order rows join `markers_test.go`;
- the twill host-neutral ban becomes one scan test over the twill stencils;
- every other row goes, the section-scoped marker-placement rows included, since `markers_test.go`'s exact marker sets pin presence;
- the `loom-discussion-spec.md` heading rows go: `internal/discussionparser`'s required list is unexported, holds only the decision record's required headings, and the parser's own tests pin it.

Rationale: a renamed Go value or a host word in a twill stencil breaks a run or another host's render unseen.
One table per kind fits `PATTERN-test-economy`'s "many items of one kind are one table-driven test".

### D3 — Where the coverage rule changes

Decision: in the same commit as the tests:

- `contracts/specs/loom-plan-spec.md` rule 3 adds: a stencil's static prose is not behavior a test covers; a stencil card carries a test only when it changes what D1 keeps.
- `contracts/stencils/loom/loom-template-plan.md`'s restatement of that rule (its "Carries the coverage for the behavior it introduces" item) carries the same exemption.
- `pattern/PATTERN-test-economy.md` gains one bullet: no test pins a stencil's static prose; D1's keep list is what a stencil test may pin, and review guards wording.
  The `PATTERN.md` one-liner is unchanged, so the PATTERN headroom is untouched.
- `loom-rubric-plan-review.md`'s **Test economy** bullet adds: a card that plans an assertion on a stencil's static prose is a finding.
- The **Test economy** lists in `loom-rubric-webster-review.md` and `loom-rubric-darn-review.md` add one item, a new assertion on a stencil's static prose, and their "behavior the diff adds with no coverage" item exempts stencil prose.
- The doc comments heading `contracts/stencils/markers_test.go`, `contracts/stencils/rubric_test.go` and `internal/websterengine/template_test.go` stop naming `claims_test.go` or prose assertions.

Rationale: the board asks for the rule to change with the tests; it lives in the plan spec's coverage rule, its plan stencil restatement, the reviewers' test-economy lists, and the claims file's own "protected, never dropped" header, which goes with the file.

### D4 — Review guards prompt wording

Decision: the webster-review and darn-review rubrics, which the Burler review row reads, gain one **Also flag** bullet: a diff that changes a stencil is read against its measuring stick (the plan card's `Intent` in webster review, the board entry in darn review); a dropped, weakened or contradicted rule the stick did not ask to change is a finding.
No new review step, agent or gate is added.

Rationale: option 2 of the board entry, chosen by the operator as the replacement guard.
This downgrades a guard from mechanical to judged: what it lets through is a prose regression the reviewer misses; what bounds it is that a loom landing passes Plan-Review and Webster-Review and a darn landing passes Darn-Review, each reading stencil changes against the stick it already uses.

### D5 — Test budget rows stay as they are

Decision: `cmd/lyx/testdata/test-budget.yaml` is not lowered for packages that lose top-level tests.

Rationale: the rows are ceilings; lowering them would fail a concurrent task that adds tests to the same package at its Merge-In, for no gain.

## Constraints

- `PATTERN-test-economy`: each moved kind is one table-driven test, not one test per row.
- The PATTERN scans over stencils (`PATTERN-parent-directive`, `PATTERN-edit-directive`, `PATTERN-role-skills-typed`, `PATTERN-no-denied-recovery`, `PATTERN-fabric-vocabulary`) stay untouched.
- `PATTERN-producer-pointer-rule`: the plan stencil's exemption matches the spec's and does not restate more of it than the stencil already does.
- `PATTERN-comment-line-breaks`: edited Go comments keep one sentence per line.
- Semantic line breaks in every edited `.md` file.

## Auto-mode assumptions

- D1's line, D2's placements and D4's measuring sticks are auto-picks; the question ledger records each.
- `docs/benchmarks/test-redundancy.md` is a measurement report and is not regenerated.

## Open risks

- `engine-registry` and `creel-core` are running and touch `claims_test.go` or template tests.
  Whichever lands second meets a modify/delete conflict on `claims_test.go`; the resolution drops the other side's rows by D1, porting a Go-value row into D2's table.
- A concurrent task that adds a prose assertion to a template or prompt test merges without conflict after this lands, reviewed under the old rubric; the next task touching that file prunes it by D1.

## Acceptance criteria

- `contracts/stencils/claims_test.go` no longer exists, and no test keeps an assertion D1 drops.
- Rewording a sentence of static prose in any stencil, leaving markers, Go-named values and twill host words untouched, fails no test.
- Renaming one of D2's Go-declared values without the stencil, or the reverse, fails D2's table test; the table is non-empty and every row names a registered stencil.
- Adding a host word to a twill stencil fails the host-neutral scan.
- Removing a marker from a stencil, swapping two order-pinned markers, or omitting a required marker from a Fill input still fails a test.
- A Go-chosen fragment or branch (webster's pack layout, burler's single-reviewer versus fork render, the friction and parent directive renders) is still told apart by a kept assertion.
- The PATTERN scans over stencils, the registry test, the rubric marker allowlist and the citation scan pass unchanged.
- The plan spec, the plan stencil, `PATTERN-test-economy.md` and the plan-review, webster-review and darn-review rubrics carry D3's and D4's text.
- `lyx gate test ./...` and `lyx gate test --tags integration ./...` are green.

## Notes for the plan writer

- `claims_test.go`'s Go-value rows are found by grepping it for `gatecli.`, `shedrun.` and `shuttleengine.`; its literal key rows include `class` and `first_card` on the webster-review rubric.
- `//lyx:guard` sits on `TestStencilClaims`; deleting it drops it from every card gate's guard step with no other change.
- `parentdirective_test.go` and `roleskills_test.go` hold PATTERN scans, which D1 keeps; a stencil reword under `creel-agents` hits them by design.
