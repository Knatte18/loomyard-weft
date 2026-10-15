# Drop stencil-prose test assertions; review guards wording

A change to a stencil's prose no longer costs a test change.
Stencil tests now pin only what Go fills, reads or names, and review guards prompt wording instead.
Wording tests were most of the weight in prompt-edit tasks with little code, where they matched or exceeded the stencil lines changed.

## What changes

- `contracts/stencils/claims_test.go` is deleted whole, including its provenance-citing protected claims and the `loom-discussion-spec.md` rows.
- The rows it held that Go depends on move into kept tests:
  - `contracts/stencils/govalues_test.go` is one table of Go-declared values (`shuttleengine.MessageTail`, `shedrun.ParkMarkerFileName`, `gatecli.SlotBusyExit`), each row naming a registered stencil.
    It also holds the twill host-neutral scan.
    Both carry `//lyx:guard`, so renaming a value outside `contracts/stencils` still fails its own gate.
  - `contracts/stencils/markers_test.go` gains a table of Go-filled marker positions, anchored on markers, headings and the end of the body only, and rows in `TestStencils_TopLevelMarkers` for stencils no other test reads in their real file.
- Template and prompt tests elsewhere (`websterengine`, `treadleengine`, `burlerengine`, `loomengine`, `orchengine`, `sheddriver`, `shedadapters`, `alignment`, `darnengine`, `loomcli`, `editdirective`, `twilldirective`, `parentdirective`, `pattern` and `cmd/lyx`) lose their assertions on static stencil prose.
  A test function left empty is deleted, and a function left with kept assertions keeps only those.
- Doc comments in kept test files no longer point at `claims_test.go` or describe prose assertions.

## The keep-or-drop rule

An assertion stays when the text it checks comes from Go or is read by Go, or when it enforces a rule across a stencil family.
That covers marker sets and Fill round-trips, values Go filled or branches Go chose, stencils naming a Go-declared constant or parsed key, and cross-stencil scans.
It goes when it checks the static prose of one stencil, whatever `why` or provenance it cites.
Literal key names with no Go definition a test can reference also go.

## Rules that change with the tests

- `contracts/specs/loom-plan-spec.md` rule 3 and the plan stencil's restatement of it exempt stencil prose: a stencil card carries a test only when it changes what the keep rule retains.
- `pattern/PATTERN-test-economy.md` gains a bullet that no test pins a stencil's static prose.
  The `PATTERN.md` one-liner is unchanged.
- The plan-review, webster-review and darn-review rubrics flag a planned or added assertion on stencil prose.
  The webster-review, darn-review and merge-review rubrics exempt stencil prose from their "no coverage" item.
- The webster-review and darn-review rubrics gain an **Also flag** bullet: a diff that changes a stencil is read against its measuring stick (the plan card's `Intent`, or the board entry), and a rule dropped, weakened or contradicted without being asked for is a finding.

## Design decisions to know

- This turns a mechanical guard into a judged one.
  A prose regression can pass when:
  - the change lands outside loom and darn (an orchestrator fix or an operator commit on `main`);
  - the Webster-Review or Darn-Review budget is 0;
  - `webster_auto_accept` is true;
  - the round reaches the hard cap.
- Assertions on text built in Go source (refusal messages, deny notices, the `shuttleengine` gate reprompt) stay, since that text is Go behavior.
- `cmd/lyx/testdata/test-budget.yaml` is not lowered.
  The rows are ceilings, and lowering them would fail a concurrent task that adds tests to the same package.
- No mechanical scan refuses a new prose assertion; review enforces the rule, as it does the rest of `PATTERN-test-economy`.
- `docs/benchmarks/test-redundancy.md` is a measurement report and is not regenerated.

## To check by hand

- `lyx gate test ./...` and `lyx gate test --tags integration ./...` were not run on the final head; run both before pushing.
- `engine-registry` and `creel-core` touch `claims_test.go` or template tests.
  Whichever lands second meets a modify/delete conflict on `claims_test.go`; resolve it by dropping the other side's rows by the keep rule, and port any Go-value row into the table in `govalues_test.go`.
- A concurrent task that adds a prose assertion to a template or prompt test merges without conflict after this lands; the next task touching that file prunes it.
- Deviations to review: the fixer-prompt row in `TestRenderPrompts` asserts `_lyx/plan` because the prompt renders the plan directory relative to the worktree root; `TestDarnSpec` has one `filepath.ToSlash(in.SpecsDir)` want instead of two; `internal/treadleengine/template_test.go` gained a small `yamlTagOf` helper.
