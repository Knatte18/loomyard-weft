# Fixer report — discussion round 2

Every finding's premise held against the code; all 11 are fixed in `_lym/discussion/decision-record.md`.
The review's line numbers sit about 14 lines past the record's; each finding was located by its decision id.

## Changes

- **F1** — D5 states that a test's markers name every struct its reflection walks, and that a test reaching `envkit.NilSeams` marks `internal/shedrecipe#Env` and each struct `Env` nests by value, recursively; the scan refuses a `NilSeams` test whose markers miss one of the nested set it derives from `Env`.
  The rationale now states what the scan guarantees, and the Bound says a directly reflecting test is checked for a marker, not for completeness.
  D5's acceptance bullet covers the missing-nested-marker case, and D6's covers a card adding a field to `landingshed.Deps`.
- **F2** — D4 resolves `pkg.Member.Sub` by its `pkg.Member` prefix as `<dir>#Member`; the Bound says the trailing field or method is not tested; the acceptance bullet adds `shedrecipe.Env.PlanIndex` as passing.
- **F3** — D6's Bound states the later-card hatch and that the earlier card's gate test run and review catch the broken test.
- **F4** — D4 names the finding for an unpartitioned ambiguous answer and a rejected answer; D8 counts an unpartitioned ambiguous answer as non-test code.
  After the self-check, the ambiguous answer has its own way forward (see Self-check).
- **F5** — fixed differently: D8 classes an unresolved member glyph as test code when it is a Rename new side whose old side is test code or its name begins `Test`, `Benchmark`, `Fuzz` or `Example`; the D8 acceptance bullet adds a card renaming a test function.
  The proposed Move new side was not added: a Move group carries no new side (`internal/planparser/plan.go`, `TargetGroup`, `Pairs` filled for Rename only; `contracts/specs/loom-plan-spec.md` Move rule).
  The proposed "Create group's file target is test code" clause was not kept: a Create member glyph names no file, so the clause classed a production function created beside a new `_test.go` file as test code.
  D8's Bound instead states that a non-prefixed helper created by member glyph counts as non-test code, so its card lists the `doc.go`.
- **F6** — D6 drops the file match from the later-card cover and limits the own-group file match to Uses, Delete, Rename and Move; the way forward says to list the test under Edit by its member glyph.
- **F7** — D5's Bound says reflection through any non-test helper, same package or other, is not seen, except `envkit.NilSeams`.
- **F8** — D9 gives the index for D6's declaring-file match and D8's classification; D4 says its method's declaring files are read by D6 and D8.
- **F9** — D4's `pkg.Member` bullet requires no whitespace and no `<`.
- **F10** — fixed differently: D4's glyph shape excludes a unit naming an existing worktree file other than a `.go` or `.cs` file, so `docs/overview.md#Modules` is not a glyph; the acceptance bullet lists a markdown anchor as untested.
  The proposed rule, that a unit's dotted last segment must end in a source extension, was not taken: a C# namespace unit such as `Norce.Models#Foo` (quarry `docs/glyph.md`) has a dotted last segment with no source extension and would stop being a glyph.
- **F11** — Auto-mode assumptions gains the D7 `||` and `untagged` bullet.

## Self-check

- fix-consistency, LOW — D5 acceptance still says "same-package helper": fixed as proposed (`_test.go` helper).
- fix-consistency, LOW — D8's new helper sentence contradicts "a test-only card is not checked": fixed with my own change, the sentence now says such a card is not test-only; the helper clause behind it was replaced (fix-proposal MEDIUM 1).
- fix-consistency, LOW — the `pkg.Member.Sub` narrowing is not in Auto-mode assumptions: fixed as proposed.
- fix-way-forward, MEDIUM — the ambiguous answer's "spell it as a glyph" way forward cannot be met: fixed as proposed, the way forward is now to cite the meant declaring file by path, which D3 tests; duplicate of fix-proposal MEDIUM 2.
- fix-way-forward, LOW — the new D5 refusal names no way forward: fixed as proposed; one sentence gives every D5 refusal its way forward.
- fix-claims, MEDIUM — a Move group has no new side: fixed as proposed, Move dropped from D8's clause.
- fix-proposal, MEDIUM — the Create-group clause classes a production member as test code: fixed with my own change, the clause removed and the helper case stated in D8's Bound, not a narrower file-based rule, since a Create member glyph names no file.
- fix-proposal, MEDIUM — ambiguous-answer way forward: duplicate, fixed above.
- fix-proposal, LOW — the derived nested set includes structs `walkNil` skips: fixed as proposed, D5 limits it to exported fields whose path `nilLegal` does not skip.
- fix-proposal, LOW — the plan-time cost of the nested set is unstated: fixed as proposed, as an Open risks bullet.

## Disputed

None.

## Deferred

None.
