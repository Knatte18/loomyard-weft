# Publish review fix-back: PR-Gate, PR-Rework and `lyx loom reject`

All 16 batches reported `status: done`, and the integration suite reported `status: OK` (HEAD `50fca39ad`).

- **Rejection record (01–02):** `landingshed` gains the `Rejection` record with its reader, writer and `RemoveRecord`. `loomengine` gains the rejection, coverage and rework-round path accessors.
- **Plan tooling (03–06):**
  - `planparser.ParsePlanFrom` parses a plan through a file-reader closure.
  - `CheckAppendOnly` compares a base and an extended plan.
  - `fabricengine.CommittedAnchoredFile` reads an anchored file as committed at HEAD.
  - `websterengine.RebaselinePlanFingerprint` restamps the plan fingerprint, with resume over an extended plan proven by test.
- **Gate and rework rows (07–12):**
  - **`PR-Gate` (`landingshed`):** the producer that owns every review decision.
  - **Rework stencil (`loomengine`):** the stencil and `ReworkSpec`, which reuses the plan role's model-spec and timeout.
  - **`PR-Rework` (`loomshed`):** appends cards, re-baselines the fingerprint and records the round.
  - **Engines and recipe:** `shedrecipe` registers the `PRGate` and `PRRework` engines. The loom recipe routes `Publish` through `PR-Gate` and `PR-Rework`.
  - **Publish split:** `Publish` returns Done at an open PR and refreshes its title and body.
- **CLI and batten (13–15):**
  - `approve` now works at the gate row and removes a pending rejection first.
  - `lyx loom reject` is new, with its refusal table.
  - Batten resumes an awaiting child on either an approval or a rejection, via `ChildDecision`.
- **Docs (16):** `docs/overview.md`, `loom-status-spec.md` and the ly-drive skill describe the gate, `reject` and the hand-made-fix rule.

Deviations from the declared file-ops:

- **Batch 10:** also edited `internal/shedrecipe/registry_test.go`, because a test pinned the registered engine names.
- **Batch 11:** also edited `internal/loomrecipe/gatequiescence_test.go`, to add `PRRework` to a writer-engine case.

No batch needed recovery, and no friction note was written.
