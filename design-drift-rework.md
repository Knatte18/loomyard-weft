# Drift compares changed values in code against a sealed expectation, with a separate judge

`drift-rework` · Task · enhancement, drift, loom, darn, stencil, config · running · Discussion-Write

## Seen (`docs/research/drift-whole-review-2026-11-04.md`)

A Drift compare is an opaque command that exits 0 or not; which outputs changed, and from what to what, exists only in the harness's log, which an agent reads.
The agent decides the scope, whether each drift was expected and whether to accept it, and an acceptance covers a whole test.
The report lists the faults this leaves (an acceptance hides later regressions in the same test, nothing checks the expectation's seal, the writer never sees the expectation) and proposes one reworked design, written against the `cs` hub's harness.

## Built since (tidy, 2026-11-05)

Task `drift-interim-repairs` landed (`83735e65`), two repairs ahead of this rework:

- `lym drift accept <test>` refuses a name that matches no file under the run's baseline profile directory (with or without the file's last extension).
  `lym drift scope --skip` stays unchecked: it runs before any capture, and a capture writes no file for a skipped test.
- The accepted profile keeps each accepted test's output as it was at accept: the verb captures the current code first, outside the run budget, and Drift-Verify and Drift-Recheck build the profile from the baseline with those kept files replacing the tests'.
  A later move of an accepted test shows as drift again.
  `RunAcceptedCapture` and the `CaptureAccepted` seam are replaced by `RunAcceptanceCapture` and the `CompareAccepted` seam; new files under `.lym/drift/` are `accepting`, `accepted-tests` and `accept.lock`.

This rework still removes the accepted profile and its seams, so these repairs live only until it is built.
Two open points from the landing's review: whether the baseline directory holds only profile files (`testFiles` in `internal/driftengine/accepted.go` admits any file there), and the partial scope itself, which goes in this rework.

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

The two interim repairs above do not replace this.

## Decisions (operator, 2026-11-05)

1. The compare moves into lym's code, under the general capture contract above, with one file per test; the `cs` hub's regression suite is small, so one file per test is fine there.
2. Open line ranges such as `12-` are allowed in the expectation for unkeyed output, without a halt, and every open range is listed in the judge's verdict.
3. Where the fix loop lives for tapestry, which depends on Webster's gate seam, is settled in the plan after reading that seam.
4. The seal is the expectation's hash, plus a check that the sealing commit precedes the writer's first commit.

The run goes autonomous: the Discussion asks the operator nothing.

## Related

`sampler-drift-trial` asks that Drift verbs work outside drift stages; this rework changes which stages exist, so that note is revisited when this lands.

## Size

A tapestry run, profile `solid`: it touches tapestry and darn recipes, Drift config keys (`PATTERN-config-reinterpretation`), stencils and records.
