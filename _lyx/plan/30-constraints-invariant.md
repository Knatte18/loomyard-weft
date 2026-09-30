# Card 30 — constraints-invariant

**Prosa:**
- `CONSTRAINTS.md`

**Uses:**
- `contracts/specs/refusal-spec.md`

**Intent:**
Record D12's invariant and the Shed Verb-Set change, so the principle outlives this audit.

Add a `## Refusal Way-Forward Invariant` section:

- It binds the modules that have a section in `contracts/specs/refusal-spec.md` (webster, shed and loom today); another module joins when its own audit adds its section.
- Every refusal reachable through a bound module's verbs names its way forward in its message, as that file defines one, and has a row in its module's section.
- A guard in a bound module that does not protect correctness warns and records rather than halts.
- A new refusal in a bound module lands with its row and its reaching test in the same commit.
- Enforcement is review discipline plus the per-row tests; there is no scan.

In the Shed Verb-Set Invariant, the generic verb list becomes `run`/`step`/`status`/`pause`/`goto`, and a new bullet states that `goto` is a history-only outcome: declared by `internal/shedengine`, returned by no producer, and accepted by every reader that validates history outcomes.

Semantic line breaks, matching the file's existing sections.
