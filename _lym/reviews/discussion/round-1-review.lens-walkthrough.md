```yaml
lens: walkthrough
outcome: reported
```

**Walkthrough lens: one concrete case per decision, run end to end, with each end state checked against the decision's purpose.**

- D1 — case: build `darn-recipe.yaml` with Record-Write added after Preflight. The record row, the writer after it on the `on_done` path, Finalize then Friction-Reflect as terminal, and `verify_source` are all present. Purpose met.
- D2 — case: `lym loom status` on a pawl seed, on a `beam` seed and on an unseeded run. Pawl is served through the exception, `beam` is refused as non-loom, and the unseeded run reads as `tapestry`. Purpose met.
- D3 — case: `lym loom insert-card` on a darn run (no `Plan-Write` row, so it refuses), and `peers` for a darn peer on another machine (falls back to the entry's `recipe` field). Purpose met.
- D4 — case 1: a darn entry whose body has H2 headings, written verbatim under `## Goal`. Partly missed (F2). Case 2: an in-flight darn run parked on Darn-Write with no record, following the halt's `goto --to Record-Write`. Missed (F1).
- D5 — case: a `tapestry` fixer drops an `Added after Discussion` entry, and the history check refuses it. Purpose met.
- D6 — case: a no-parent darn writer meets an outdated entry and writes a stop report with no code commit. The handoff gates run before the no-commit check. Missed (F3).
- D7 — case: a darn Code-Review round with the snapshot gone. The record's `## Goal` holds the body verbatim, plus its added entries. Purpose met.
- D8 — case: a pawl run's verify source when pawl has no `verify_source` key. Covered by the D2 exception. Purpose met.
- D9 — case: a `tapestry` Finalize reroute goes Parent-Align, Drift-Verify, Code-Bouncer, Describe. Purpose met.
- D10 — case 1: a fresh `tapestry` seed with a broken `batcher.yaml` fails at Preflight. Purpose met. Case 2: a run already blocked on Tapestry-Preflight by `batcher.yaml`, resumed after landing. The check is skipped (F4).
- D11 — case: a seed recording `beam`. `lym shed` refuses it as unknown, and `lym loom` refuses it as non-loom. Purpose met.
- D12 — case: the `solid` profile's tapestry block `loom.plan_max_bounces: 2`. `moveRunSettings` puts it in `tapestry`'s local document. Purpose met.
- D13 — case: a profile's tapestry block `{shed: {driver_kind: go}}`. It reaches `PrepareRunSettings` and the shed keys are `CategoryNone` (`internal/sheddriver/config.go:77-84`), so the local layer admits them. Seed-Child writes it, and `childDriverKind` reads the task location's dirs (`internal/battencli/wire.go:1319`). Purpose met.
- D14 — case: `lym shed status` on a batten run. Unchanged. Purpose met.

**F1 — MEDIUM, design — D4, in-flight darn runs: following the halt's own way forward fails to resume the run.**
- Darn-Write's `discussion` gate runs at the handoff, after the writer session. So an in-flight run with no record spends a full writer session first. The writer may commit code, and Darn-Write commits on every non-empty pointer (`internal/loomshed/darnwrite.go:67-92`).
- The gate's finding is then the generic "decision record does not exist" (`internal/discussionparser/validate.go:78-80`), and the Stuck reason gains only `darnResumeWayForward`. Nothing in the existing mechanism produces the `goto --to Record-Write` reason D4 claims. D4 never says which code adds it.
- Following that `goto` anyway sends Record-Write's `on_done` back to Drift-Baseline. Drift-Baseline's precondition is that HEAD equals the branch point and the tree is clean (`internal/driftengine/doc.go:29`). Once the writer has committed, that refuses whenever drift is enabled, so the run never resumes.
- Evidence: `contracts/recipes/darn-recipe.yaml` row order and the Darn-Write gates; `internal/driftengine/doc.go:29`.

**F2 — MEDIUM, design — D4: copying the body verbatim breaks the record's section structure.**
- Board bodies carry their own H2s. This task's own body has `## Seen`, `## Three layers`, `## The decision record` and `## Open`.
- Pasted verbatim under `## Goal`, these become extra top-level record sections. The spec pins "these H2 sections … and no others besides the two optional ones" (`loom-discussion-spec.md`, "The decision record").
- Any reader that takes the `## Goal` section stops at the body's first H2. A body H2 named `## PATTERN changes` or `## Decisions` would be parsed as the record's own section.
- The validator accepts the extra H2s (`internal/discussionparser/validate.go:64`), so nothing catches it.
- D4's bound covers only an empty body. It says nothing on demoting or fencing the body's headings.

**F3 — MEDIUM, design — D6: the stop report reaches the no-commit check only after the handoff gates have reshaped it.**
- Darn-Write's handoff gates run in order: `discussion`, `verify`, `description`, each with 3 attempts (`contracts/recipes/darn-recipe.yaml`, Darn-Write config).
- With D4's record in place, the discussion gate now passes. A stop report written as the change description then meets the `description` gate, which validates it against the final-summary format (`contracts/specs/loom-final-summary-spec.md`, "Format and validation"). If the report does not conform, the writer is re-prompted until the gate exhausts. The run then halts with that gate's findings as its reason, not with "the change description as the halt's reason" D6 promises.
- The stencil's current instruction, "end your turn with the stop report unchanged, so the gate exhausts" (`contracts/stencils/darn/darn-template-write.md`, Step 3), points at the discussion gate, which no longer fails.
- D6 never says where the no-commit check sits relative to the gates, or what form the stop report must take to pass `description`.

**F4 — LOW, design — D10: a run blocked on Tapestry-Preflight resumes past the `batcher.yaml` check that blocked it.**
- The retired-row map sends a run parked on Tapestry-Preflight to Discussion-Write. That run has already passed Preflight.
- So a run that was blocked by a broken `batcher.yaml` resumes with no `batcher.yaml` check until the Webster row. That is the late failure D10's rationale exists to prevent.
- Evidence: `internal/loomshed/tapestrypreflight.go` `checkBatcher`; the `tapestry-recipe.yaml` rows Preflight → Tapestry-Preflight → Discussion-Write.

Tool calls:
1. `tapestry-recipe.yaml` row order, `on_done` and `subject`, and the discussion parser's H2 handling: I needed the tapestry edges for D9 and D10, and the parser rules for D4's verbatim body.
2. `internal/sheddriver/config.go` categories: to check that a profile block under `shed` passes the local-layer check (D13).
3. The final-summary spec headings and the `childDriverKind` caller: for the stop report in D6, and which dirs the child's driver kind is read from (D13).
4. `internal/discussionparser/validate.go:40-120`: to confirm that extra H2s are not flagged (D4).
