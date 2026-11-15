```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set, run for F1 (every site offering a hand `dotnet test` run) and F4 (`CodeLocation` branches). I checked every finding.

**F1: no findings.**
- **Sites offering the hand run:** the F1 search over `dotnet test` and "by hand" found only these three:
  - `contracts/stencils/darn/darn-fragment-make-change.md:16`
  - `contracts/stencils/webster/webster-body-implementer.md:118`
  - `internal/gatecli/tierrun.go:378` (the `noTestNotice` way-forward clause)

  The fix gates all three on `refuse_raw`.
- **Docs:** `internal/gatecli/doc.go` was also updated.
- **Other contract text:** `contracts/stencils/loom/loom-block-plan-cards.md:63`, `contracts/specs/loom-plan-spec.md:352` and `loom-rubric-plan-review.md:110` say "never a raw test command". They concern `**Verify:**` lines, not the hand run, so they stay consistent.
- **Hub doc:** `docs/dotnet-hub.md:170-173` shows `refuse_raw: ["dotnet test"]` and says the guard steers agents to `lym gate test`. It does not contradict the gated wording.

**F4: no findings.** `CodeLocation` has one production caller, `internal/gatecli/run.go:673`. The new `internal/fabricengine/codelocation_test.go` covers both unchanged-return branches: not a weft sibling, and no paired code directory. The success branch that calls `NamedWorktree` is exercised by the existing `TestGateTestMap` records-sibling rows.

**Unchecked:** F2 and F3 are not set-premise findings, so I ran no search for them. F2's fix is a documented limit plus line wording, not detection, which is the fixer's call to record as a different fix.

Tool calls:
1. Grep over `.md`, `.go` and `.yaml` for the hand-run, raw-test and `dotnet test` phrases, to list every site that offers or bans the hand run. My context held only the files I had read, not the whole tree.
2. Grep for `CodeLocation(` callers, to see its call sites and test coverage. My context showed its definition and the `run.go` diff but not the other callers.
