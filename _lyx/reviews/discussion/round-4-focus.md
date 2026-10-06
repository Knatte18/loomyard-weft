---
round: 4
focus:
  - "Decision 12 in `_lyx/discussion/decision-record.md` defines the per-package coverage figure by filtering the coverage profile by directory. Settle whether the definition is unambiguous for the nested in-scope packages the Scope names (`internal/gitrepo/internal/gitoracle`, `internal/reedengine/render`, `internal/shuttleengine/claudeengine`): exact directory or prefix match, how statements are counted, which files are excluded, and which `-covermode` the profile uses."
  - "The decision added after Discussion that merges scenario tests into ordered steps merges a family only when it cuts wall time, and Decision 11 measures wall time per package, before on the base commit and after on the final tree. Settle whether a measurement exists that can decide one family's merge while editing, given the load noise the Open risks name."
  - "The same scenario decision runs steps in order on one shared fixture, where separate tests each had a fresh one, and Decision 8 re-runs a test alone with `-run` to tell a real order dependence from a tool artifact. Settle what bounds a failed step's effect on later steps, which tests may be merged, and how Decision 8's alone-rerun applies to a step."
---

Each entry names a site in `decision-record.md` and the question to settle.
Check any instance against the rubric's `Do not flag` list and the writer stencil's symmetry rule before raising it.
