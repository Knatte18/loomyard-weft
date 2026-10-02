---
round: 1
ledger:
  - key: strand-name-retirement-mechanism
    rounds: [1]
    status: resolved
  - key: session-name-provider-degradation
    rounds: [1]
    status: resolved
  - key: ly-hub-workflow-doc-home
    rounds: [1]
    status: resolved
  - key: worktree-parent-seed-self-alias
    rounds: [1]
    status: resolved
  - key: support-log-superseded-hub-position
    rounds: [1]
    status: resolved
  - key: criteria-bad-slug-refusal-and-package-docs
    rounds: [1]
    status: resolved
  - key: redundant-hand-started-seed-criterion
    rounds: [1]
    status: resolved
  - key: dated-deliberation-in-rationales
    rounds: [1]
    status: resolved
  - key: standalone-mode-naming-undecided
    rounds: [1]
    status: open
---

Round 1's reviewer raised eight findings, F1–F8, and approved.
The fixer applied all eight in place, and the current artifacts reflect each fix.
That covers the `strand_name` retirement behaviour, the provider drift cases and degradation, the hand-started hub path as a non-deliverable, the default-run seed at `_lyx/shed/<slug>/seed.json`, the support log against the hub review, the two missing criteria, the redundant seed criterion and the dated attributions.

The judge raised one open finding: the record never decides how strands are named in standalone mode (`lyx webster`/`burler` with `--target-dir`).
Standalone mode boots its own reed session through `standalonegeom.ReedGeometry`, with no weft, no `.lyx-code`, no fabric slug and no shed seed.
The record makes the code mandatory for every reed strand, and it names `hubgeom.ReedGeometry` as the only teller.
