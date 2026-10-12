---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: _lyx/discussion/decision-record.md (Scope, In)
    summary: run-robustness also changes Go package internal/websterengine and its doc.go, which this run's Scope edits
    fix: The run's parent decides the order of the two landings, or the split of internal/websterengine/doc.go between them, before this run passes parent-review; record that decision in this run's decision record under Constraints.
---

### [BLOCKING] Package overlap with run-robustness in `internal/websterengine`

**Location:** `_lyx/discussion/decision-record.md`, Scope In: `internal/websterengine/doc.go`, `internal/websterengine/template_test.go`, `internal/websterengine/beginbatch_test.go`.

**Issue:** Peer `run-robustness` (running · Webster) changes the same Go package.
Its decision record puts `internal/websterengine`'s verify gate in scope (D5, D6: no flake rerun after a hang, in `verifygate.go`) and has `websterengine`'s package doc describe D5 and D6; its plan's card 12, `webster-hang-gate`, carries that change.
Its branch does not touch `internal/websterengine` yet, so the shared `doc.go` edit is still ahead of both runs.
The two changes describe different behaviour (pack-on read texts here, the verify gate there), so the overlap is the package and its `doc.go`, not a contradicting decision.

## Scope check

The design matches the board entry `kickstart-read-all-first`:

- First turn reads every card file and every span under every card, in parallel, with `offset` and `limit`, before any edit: Decision 1.
- The whole-file rule for files an earlier card edited goes: Decision 4.
- The pack stays grouped per card: Scope Out keeps its content, grouping and rendering, and a criterion pins it.
- The fork still reads further when a card needs it: Decision 5 keeps that sentence.
- The operator's alternative (per-card quarry resolve) is not built; the support log rejects it with a reason, and the entry offers it only as an alternative.
- Decision 2 moves the existing reads of uncovered files and unresolved glyphs ahead of the first edit, batch-wide.
  The record labels it an extension; it changes read timing only, which the entry says is the one thing that changes, and no entry rules it out.

No board entry is duplicated or contradicted.
`kickstart-pack-measure`'s Next audit waits on this task landing; `kick-start-pack`, `enclose-kickstart` and `weaver` stay out of scope.

## Other peers

- `lyx-halt`: its decision record lists `contracts/stencils/claims_test.go` claims that pin pause lines and webster package docs among its same-commit edits, and names `internal/websterengine/pause.go`.
  Its branch has touched neither `claims_test.go` nor `internal/websterengine` so far, and its plan overview names no file there.
  If it does edit them, it shares `contracts/stencils/claims_test.go` with this run and possibly `internal/websterengine/doc.go`; the parent's decision on F1 should cover it too.
- `convergence-plan`: names `websterengine/gitwrap.go` as an existing reader only; no overlap.
- No parent commits are listed.
