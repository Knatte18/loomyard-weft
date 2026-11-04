# Drift compares cell by cell in code against a sealed expectation, with a separate judge

`drift-rework` · Note · enhancement, drift, loom, darn, stencil, config

## Seen (`docs/research/drift-whole-review-2026-11-04.md`)

A Drift compare is an opaque command that exits 0 or not; the drifted cases, lines and values exist only in the harness's log, which an agent reads.
The agent decides the scope, whether each drift was expected and whether to accept it, and an acceptance covers a whole test.
The report lists the faults this leaves (an acceptance hides later regressions in the same test, nothing checks the expectation's seal, the writer never sees the expectation) and proposes one reworked design.

## Idea: the report's design

Build the design in the report's "Design" section:

- The hub's `capture` writes one text file per case, one line per quantity; lym's code diffs the baseline against the current capture, cell by cell (case, line, old value, new value), and the compare command becomes optional.
- A Drift-Expect agent, after the capture, writes a structured `expected.yaml` (lines or ranges per case, added or removed cases, one reason each), sealed when the writer starts.
- Containment by code: a changed cell outside the expectation is a regression and stops the row.
- A separate judge agent rules each expected, changed cell `matches` or `does not match`; an accepted cell is stored as its exact tuple, and only a cell whose tuple changes is judged again.
- The writer gets the containment check as a gate entry of its own session.
- Removed: `partial`, `skip`, `skip_file`, `accept`, `withdraw`, `waive`, the accepted profile and its seams, the run budget, side capture and `DriftRecheck` as a separate engine.

The two interim repairs (`drift-interim-repairs`: a name check on `lym drift accept`, the accepted profile taken at accept) do not replace this.

## Open decisions (operator)

The orch's recommendation stands beside each; none is decided.

1. Move the compare into code. Recommended: yes.
2. Open ranges such as `12-` in the expectation. Recommended: allowed without a halt, but listed in the verdict.
3. Where the fix loop lives for tapestry, which depends on Webster's gate seam. Recommended: settled in the plan after reading that seam.
4. Seal strength. Recommended: the hash, plus a check that the sealing commit precedes the writer's first commit.

## Related

`sampler-drift-trial` asks that Drift verbs work outside drift stages; this rework changes which stages exist, so that note is revisited if this is built.

## Size

A tapestry run, profile `solid`: it touches tapestry and darn recipes, Drift config keys (`PATTERN-config-reinterpretation`), stencils and records.
