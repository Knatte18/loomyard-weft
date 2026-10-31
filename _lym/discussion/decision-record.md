# Decision record: second recurring test-time and growth audit

## Goal

Append the second recurring audit's section to `docs/benchmarks/test-suite-timing.md`, measured by the first audit's method (`31d9ae96`, 2026-10-28) so the walls and counts compare.
The change is a docs change only; no code, test or PATTERN entry changes.

## Scope

- Twelve gate runs and no more: `lym gate test --tier N --all -- -count=1` for `tier0`, `tier1` and `tier2`, each as one cold run (discarded) then three warm runs, the median of the warm three being the tier's wall.
- A new `## test-time-audit-2: second recurring audit` section after the first audit's section, with its Correction block, holding:
  - the walls per tier against the 2026-10-28 audit;
  - top-level test counts per tier and the packages that grew or shrank most since then;
  - what `test-prune-audit-leads` (landed 2026-10-29) changed in counts and walls;
  - prune candidates as lines; no board task is filed.
- The load and quiet-machine checks as the first audit recorded them, and any run in flight on the machine during the measurement.
- Out of scope: other tiers (`tier3`), repeat runs beyond the twelve, any test or code change, any board task.

## Decisions

### D1 — Same route and method as the first audit

Every wall is the gate's summary-line wall for `lym gate test --tier N --all -- -count=1`, with the worktree held dirty by one untracked file during the measurement so the gate reads no green record, and the file removed afterwards.
Rationale: the first audit's numbers came this way, and `go run ./cmd/testtiming` is refused to an agent session.

### D2 — Per-package counts come from `go test -json` events keyed by package

The first audit's Correction block found that the gate-log split by package was unreliable and told a later audit to count from `go test -json` events keyed by package or from a verbose run of the package alone.
Tier totals still come from the gate's logs as before; the per-package growth table uses the keyed counts, produced by the same gate-free `go test -json -count=1 -tags tierN` listing of top-level `--- PASS|FAIL|SKIP` events, so the totals and the split agree.
The listing is not a timed run and does not count among the twelve.
Rationale: the Correction block's instruction, and a comparison table built on a split known to mislead would repeat the first audit's error.

### D3 — The baseline for growth is the first audit's per-package counts, not the older report

The first audit's per-package split is known unreliable (its Correction block), so growth is stated per package only where a recount of both the first audit's revision (`ba2a346b`) and today's tree is available from `go test -json` events; otherwise the per-package line says the baseline is unreliable and names the `func Test` declaration count.
Rationale: avoids pinning a comparison on figures the doc itself retracted.

### D4 — `test-prune-audit-leads` is read from git

The task landed as commit `4a059d2a` ("Run reedengine tier2 tests in parallel; fold one battenrecipe duplicate"); its effect on counts and walls is stated from that commit's diff and from the audit's `reedengine` `tier2` and `battenrecipe` figures, not from a new run on the pre-commit tree.
Rationale: the twelve-run limit forbids a before-tree measurement; the commit's diff plus the two audits' figures bound its effect.

### D5 — Machine state is recorded and in-flight runs are named

Before the first run: `lym tender status`, `lym gate status`, the load average.
After the last: the load average.
Other sessions' runs on the machine during the measurement are named in the section, and a run disturbed by another gate run is reported, not discarded silently; it is repeated only if it breaks the cold-then-three-warm structure, which the twelve-run limit counts as the same run's retry.
Rationale: the task requires naming in-flight runs; a gate slot wait makes the walls machine-dependent.

## Constraints

- The section is a measurement report in the same style as the first audit's, with the same "records one run and is not meant to stay true" note.
- `PATTERN-test-speed`: tests run only through `lym gate test` by gate key; no raw `go test` run for timing.
  The `go test -json` listing in D2 is read-only counting and is run through `lym gate test` too where the gate allows it; otherwise the count falls back to `func Test` declarations and the section says so.
- No tier beyond `tier0` to `tier2` is run.
- The prose follows `scribe:prose`: one sentence per line, no pinned perishable specifics outside the measurement report.

## Auto-mode assumptions

- No operator is present; the board entry is the whole statement of the task and no decision note answers it.
- The entry's dependency `adopt-check-visible` is done and landed in the tree.
- The two other runs in flight at the start (`landing-watch` at Worktree-Create, `run-verbs-shared` at Discussion) are expected to load the machine lightly and are named in the section.

## Open risks

- A concurrent gate run or build on the machine can lift a wall; the spread within a tier's three warm runs is the check, and the first audit's spread was under 0.3 s per tier.
- The gate may carry a run of an unchanged green tree as a record taking 0.0 s; D1's untracked file prevents it.
- A flaky failure in a measured run would not fit the cold-plus-three structure; it is reported with the failing test, and the run is retried once.

## Acceptance criteria

- The new section holds a YAML state block shaped like the first audit's, with machine, OS, Go version, revision, both load averages, per-tier test counts and per-tier walls (D1, D5).
- The section lists the four runs per tier, the discarded cold one named as such, and states that all twelve passed or names the failures (D1, D5).
- The walls table sets each tier's wall against the 2026-10-28 audit's wall (D1).
- The count table sets each tier's top-level test count against the first audit's, and the per-package growth and shrink tables come from keyed counts or say why they cannot (D2, D3).
- A part of the section states what `test-prune-audit-leads` changed in counts and walls (D4).
- Prune candidates are lines, each saying it is a lead and not a verdict, and no board task is filed (D2, D3).
- The worktree is clean of the measurement's untracked file at commit (D1).
- `lym gate test --wait` passes on the commit, and `lym loom validate-description` is clean.
