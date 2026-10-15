# Support log — plan-test-file-path

## Interview

Autonomous run: no operator and no advisors; each turn is the writer's own question with its auto-pick.

### Turn 1 — Does a `plan:` handle carry the file?

Read `loom-plan-spec.md`'s "Plan: handles" section and `handle-malformed`: the handle's unit half must be the package directory under `language: go`, never a `.go` file.
The board entry's premise "the declaration carries the file" is therefore wrong.
Auto-pick: the plan does not pin the file; the fork chooses it in the handle's package, as for production members (D1).

### Turn 2 — How does the fork learn where neighbouring tests live?

Read the webster fork fragments and `planindex.KickstartPack`: handles appear in the pack's unresolved list and are never located; a plain path led to a whole-file read.
Auto-pick: a card names the existing tests, helpers and fakes its new tests reuse under `**Uses:**`, so the pack resolves their spans and file (D2).

### Turn 3 — Where and when does the gate refuse?

Read `planglyph/index.go`: `card-fabric-reference` is a `planparser` function the index appends at format and rework validation, not dispatch.
Auto-pick: the same placement, skipping done cards, so an in-flight plan approved under the old rule never halts at dispatch after a deploy (D3, D4).

### Turn 4 — Exemptions and edge cases

Considered `language: none`, `testdata` fixtures named `_test.go`, a header-only change such as a build constraint, and the non-`**Edit:**` groups.
Auto-pick: skip `language: none`, exempt `testdata`, a header-only change names the file's members, other groups stay legal (D3, D5, Scope).

### Turn 5 — Does Webster need a change?

Read `planglyph/scope.go`: `ScopeGuard` covers a symbol-less change in a target's package directory, and a handle contributes its expected glyph.
Auto-pick: no Webster change (D6).

### Turn 6 — Lens review by forks

All six forks returned.

- goal-scope MEDIUM, Goal hides that the card names no file for a new test: fixed, Goal says so.
- goal-scope LOW and leanness LOW, D2 unlicensed: rejected; D2 is the answer to the board's third bullet, and its rationale now says so.
- goal-scope LOW, stencil's paired "Create test file only for an uncovered surface" rule not in Scope: fixed, Scope names both paired rules.
- goal-scope LOW and leanness MEDIUM, D6 contradicts the ScopeGuard open risk: fixed, D6 limits its claim to internal-test handles and points at the risk.
- goal-scope LOW, testdata exemption and done skip unasked: no change, the lens confirmed both bounded.
- pattern-fit MEDIUM, no `refGate` ledger entry for the new ref-kind decision: fixed in D4.
- pattern-fit MEDIUM, `_test.go` and `testdata` tests read the unit as a path: fixed, D4 routes through `Glyph.UnitPath`, and Scope lists the spec sentence that gains this reader.
- pattern-fit LOW, premises LOW and scenarios LOW, done skip at rework undefined and unlike the precedent: fixed, D3 skips done cards at the format gate only and states why rework needs none.
- pattern-fit LOW, D5 and D2 pattern fit: no change, the lens found no departure.
- premises LOW, external-test handle ScopeGuard mapping unverified: kept as an open risk, reworded to say what is known.
- scenarios MEDIUM, no criterion for the rework gate path: fixed, criterion added.
- scenarios MEDIUM, `<dir>_test` handle binding unverified: fixed; `internal/planglyph/glyphchain_test.go` already binds one, noted for the plan writer.
- scenarios LOW, re-sign arrow on a test file reports two findings: rejected; distinct checks reporting one bad entry twice need no pinning.
- scenarios LOW, plan-text-only or disk-reading: fixed, D3 says plan text alone.
- scenarios LOW, suffix over-match and non-test paths: fixed, criterion added.
- leanness LOW, D1 premise narrative: fixed, moved to Turn 1 only.
- leanness LOW, Webster no-change said twice: fixed, Scope points at D6.
- leanness LOW, duplicated auto-mode bullets: fixed, cut.
- leanness LOW, D3 padding bound sentence and D4 repeated dispatch reasoning: fixed, both trimmed.
- leanness LOW, Goal restates Scope: fixed, cut.
- leanness LOW, repo-wide constraints restated: fixed, the stencil-ownership and doc bullets cut; the doc rule stays in Scope's list of what changes.
- board-fit LOW, wording-tests bullet reads as leaving broken tests: fixed, Scope takes in every test this task breaks.
- board-fit LOW, `webster-deviations-list` unnamed: fixed in D6.
- board-fit LOW, pack audit commit neither in nor out: fixed, Scope Out.

## Rejected alternatives

- Let a handle's unit name the `_test.go` file: refused by `handle-malformed` for a reason that still holds, since `quarry.Resolve` never answers a file-unit member ID.
- Keep the plain test-file path beside the handles: that is the status quo the board entry removes.
- List the test file under `**Uses:**`: contradicts `**Uses:**`'s meaning, since the card changes the file.
- Run the check at dispatch too: halts in-flight runs whose plans were approved under the old rule, where the writer can no longer act.
- Gate every `.go` file path under `**Edit:**`: not asked for; YAGNI.
- Put the check in `internal/planglyph`: classifying a ref's shape is `planparser`'s alone (`PATTERN-ref-shape-registry`).

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): does the plan pin the test file? No; the fork picks it inside the handle's package (D1).
- Q2 (resolved, auto-pick): how does the fork find neighbouring tests? Through `**Uses:**` member glyphs (D2).
- Q3 (resolved, auto-pick): which gates run the check? Format and rework validation of the plan index, done cards skipped, never dispatch (D3).
- Q4 (resolved, auto-pick): exemptions? `language: none` and `testdata` paths only (D3).
- Q5 (resolved, auto-pick): does Webster change? No (D6).
- Q6 (resolved, auto-pick): does a handle on an external test unit bind? Yes, an existing chain test binds one (D5).
- Q7 (open, informational): does `ScopeGuard` map a handle's `<dir>_test` unit to its directory? Left as an open risk.
