# Drift compares changed values in code against a sealed expectation, with a separate judge

`drift-rework` · Note · enhancement, drift, loom, darn, stencil, config

## Seen (`docs/research/drift-whole-review-2026-11-04.md`)

A Drift compare is an opaque command that exits 0 or not; which outputs changed, and from what to what, exists only in the harness's log, which an agent reads.
The agent decides the scope, whether each drift was expected and whether to accept it, and an acceptance covers a whole test.
The report lists the faults this leaves (an acceptance hides later regressions in the same test, nothing checks the expectation's seal, the writer never sees the expectation) and proposes one reworked design, written against the `cs` hub's harness.

## Idea: the report's design, made general

Drift stays a general lym module: nothing in it assumes one hub's harness or output format.

### What a hub's capture owes

- The hub's `capture` command writes deterministic text files under `{baseline_dir}`: the same code gives byte-identical files.
- One file per case (a test, a scenario, a fixture); any text a line diff can read, such as golden files, snapshot output, CLI output or pretty-printed JSON.
- A line that carries a key (`name = value`, `name: value`, or a pattern the hub declares in `drift.yaml`) is identified by its key; a line without one by its line number.
  Keys keep the compare stable when a change inserts or removes lines, so a hub with keyed output needs no line ranges.
- A hub whose capture cannot meet this keeps today's opaque compare with whole-test judgment, chosen in `drift.yaml`.

### The compare and the expectation

- lym's code diffs the baseline capture against the current one per case, into a set of changed values: (case, key or line, old value, new value), plus cases and lines added or removed.
  The compare command becomes optional.
- A Drift-Expect agent, after the capture, writes a structured `expected.yaml`: per case the keys or lines it expects to change, the cases it expects added or removed, and one reason each; sealed when the writer starts.
- Containment by code: a changed value outside the expectation is a regression and stops the row.
- A separate judge agent rules each expected changed value `matches` or `does not match`; an accepted value is stored as its exact tuple, and only a value whose tuple changes is judged again.
- The writer gets the containment check as a gate entry of its own session.
- Removed: `partial`, `skip`, `skip_file`, `accept`, `withdraw`, `waive`, the accepted profile and its seams, the run budget, side capture and `DriftRecheck` as a separate engine.

The two interim repairs (`drift-interim-repairs`: a name check on `lym drift accept`, the accepted profile taken at accept) do not replace this.

## Open decisions (operator)

The orch's recommendation stands beside each; none is decided.

1. Move the compare into code. Recommended: yes, under the general capture contract above.
2. Open line ranges such as `12-` in the expectation, for unkeyed output. Recommended: allowed without a halt, but listed in the verdict.
3. Where the fix loop lives for tapestry, which depends on Webster's gate seam. Recommended: settled in the plan after reading that seam.
4. Seal strength. Recommended: the hash, plus a check that the sealing commit precedes the writer's first commit.

## Related

`sampler-drift-trial` asks that Drift verbs work outside drift stages; this rework changes which stages exist, so that note is revisited if this is built.

## Size

A tapestry run, profile `solid`: it touches tapestry and darn recipes, Drift config keys (`PATTERN-config-reinterpretation`), stencils and records.
