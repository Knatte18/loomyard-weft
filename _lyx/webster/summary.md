# Verify triage: integration-stage regression triage

All 12 cards reached `done`: integration report type, verify output parser, verify capture, integration fork log, triage core, bisect by identity, summary triage sections, triage recording, stage wiring, stencil verdict handoff, triage docs, and rebegin-keeps-start-sha.
The integration stage now runs triage (rerun and baseline classification, identity-based bisect) and demotes a done result when a regression is found.
The summary and stencil carry the triage sections, and `begin-batch` keeps a batch's recorded start SHA across a re-begin.

An earlier integration report for this plan said `FAILED`, at head `f1b58dad0`.
Since then the tree advanced to `31fa26145`, which includes the batten fixes.
I re-ran the plan-level `## verify:` command through one integration fork, and it reported `status: OK` at `31fa26145` with no deviations.
No card reported deviations from its declared file-ops.
