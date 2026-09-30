# Card 29 — package-docs

**Prosa:**
- `internal/websterengine/doc.go`
- `internal/shedengine/doc.go`
- `internal/shedverbs/doc.go`
- `internal/loomshed/doc.go`

**Uses:**
- `contracts/specs/refusal-spec.md`

**Intent:**
Bring the package documentation of the three bound modules in line with this task, linking the refusal table rather than restating rows (D3).

- `internal/websterengine/doc.go`: a new section describing D4–D9 as built — the correctness/policy split, the once-per-identity ledger, the verify re-run behind a policy warning, a batch failed on its merits with recover-batch as the way forward, an unattributable report archived and re-driven, run-exit dispositions, and drift about a later card as a warning — plus `lyx webster rebaseline` in the plan-staleness guard section, and one line pointing at `contracts/specs/refusal-spec.md` for every refusal's way forward.
  Fix any existing sentence the new behaviour falsifies, such as a claim that every audit violation fails the call.
- `internal/shedengine/doc.go`: the history-only `goto` outcome, the episode boundary it sets for its segment, the way-forward text on the lookup and budget stops, and the refusal-table link.
- `internal/shedverbs/doc.go`: `goto` among the generic verbs.
- `internal/loomshed/doc.go`, the loom package that owns its gate rows: one paragraph linking the refusal table for loom's gates and naming `goto` as Loom-Preflight's half-finished way forward.

Comments use semantic line breaks.
