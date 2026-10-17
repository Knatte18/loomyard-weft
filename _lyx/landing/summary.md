# Drop refusal-spec.md's per-refusal table, keep the way-forward rule

`contracts/specs/refusal-spec.md` grew with every new refusal: each table row quoted the `way forward:` clause its message already carries, and `PATTERN-refusal-way-forward` required a row per refusal.
The spec now holds only the contract every refusal message follows, and the message plus the test that reaches it are the record.

## What changed

- `contracts/specs/refusal-spec.md` keeps `## What a way forward is` and `## Classes`, reworded so nothing refers to a row, a cell or a section.
  A new `## Reaching tests` section states that every refusal has a reaching test, with two exemptions: a propagated raw I/O failure, which is transient, and a fail-loud wiring guard.
  These are the two exemptions the Plan stencils cite, formerly worded as the grouped transient row and the grouped wiring-guard row.
  The per-module sections (webster, shed, loom, gate, batten lock, pause, hibernate) and `## Out of scope` are gone.
- `PATTERN-refusal-way-forward` drops the row-per-refusal clause in `PATTERN.md` and its background.
  The background now names its bound modules (webster, shed, loom) directly, since "the modules with a section" no longer names anything.
  `pattern/PATTERN-no-denied-recovery.md` and the scan's comment in `cmd/lyx/norecoverydeny_test.go` refer to that binding instead; the scanned prefixes are unchanged.
- The shed driver stencil tells the driver to quote the refusal message's own `way forward:` clause rather than look the refusal up in the table.
  Its pointer at the spec goes, and so does the matching `citationAllowlist` entry in `contracts/stencils/citation_enforcement_test.go`, which would otherwise fail as stale.
- `loom-template-plan.md` and `loom-rubric-plan-review.md` cite the two exemptions as the spec now states them.
- The `doc.go` mentions in `websterengine`, `shedengine`, `loomshed` and `loomcli`, `docs/overview.md`, `webster-spec.md`'s see-also line and two Go comments no longer refer to the table or its rows.

## Design decisions

- The webster section's fork-audit severity preamble is dropped rather than moved, because `internal/websterengine`'s `doc.go` already documents the correctness/policy split.
  The note that a validation verb's findings envelope and go's own exit code from `lyx gate test` are not refusals moves into `## Classes` in one sentence.
- No refusal message or its `way forward:` clause in Go code changes.
- A driver deployed from an earlier stencil still points at the spec's path, which still exists, and finds no rows there, so it reports refusals verbatim until the next deploy.
