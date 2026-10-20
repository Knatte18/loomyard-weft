---
verdict: APPROVED
---

Alignment at `pr-gate` for `fork-findings-wait-all`, PR 781 at `d31f3aced483fa32d2d1174d898732a9c6a7e790` against `main`.

## Scope against the board entry

The decision record carries no `Added after Discussion` entries, so the board entry is the measure as written.

- The entry asks both named stencils to state the hard rule: edit nothing until every fork has returned, then judge all findings together, then edit.
  `contracts/stencils/twill/twill-directive.md` and `contracts/stencils/burler/burler-fix-self-check.md` each add that sentence to step 3.
- The entry asks for the same check on any other stencil where a session spawns review forks and acts on their findings.
  The only other such stencil is `contracts/stencils/burler/burler-cluster-fan.md`, whose handler consolidates fork findings into the review file; it now writes nothing to that file until the last fork has returned.
  The Discussion lead and the Plan writer render the twill directive through `internal/loomengine`, so they carry the rule without a stencil of their own.
  The remaining fork-spawning stencils are not review forks acted on by an editor: Webster's implementer forks, the shed driver's stop-report fork and the orch's one-shot parent-review fork.
- The package docs `internal/twilldirective/doc.go` and `internal/burlerengine/doc.go` describe the changed procedures, as the docs-in-the-same-commit rule asks.
  The burlerengine edit also reflows a fixed-column comment block into semantic line breaks around the changed phase (3); that is the touched text, not added scope.

No change goes beyond the entry, and no point of it is left out.

## Open risks

The record's one open risk, that the rule is prompt text with no gate checking it, names no parent correction and carries no escalation.

## Overlap

`lyx loom peers` lists no landed parent commits.

- `origin-reconcile` (row 9) and `run-reentry` (row 7) list no changed paths, and their subjects do not touch lens-fork review procedure.
- `lenses-template-frozen` (row 0, unknown order) changes `internal/lenses/doc.go`, `internal/lenses/template.yaml` and config reconcile code; this run changes neither the lens library nor its config, so there is no overlap.

No overlapping peer or parent commit, so no landing order applies.
No other board entry is made obsolete by this run.
