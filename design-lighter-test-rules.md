# Lighter test rules for small changes

`lighter-test-rules` · Task · enhancement, stencil, webster, burler, loom

## Idea (operator, 2026-10-13)

The project's test and doc rules make small changes large.
`webster-pack-last` moved the kick-start pack into a file of its own: about 70 lines of change, about 140 lines of tests and about 20 lines of required docs and registration, 14 files in all.

## Seen

- Tests that pin stencil wording (`contracts/stencils/claims_test.go`, the template tests such as `internal/websterengine/template_test.go`) change with every sentence a prompt changes, and prompt changes are much of today's work.
- The doc rule (`doc.go` in the same commit) is cheap and keeps the package docs true.

## Options to discuss

1. Tests pin only what must hold: markers, file paths, order and the presence of load-bearing rules, not the wording of sentences.
2. Stencil wording claims move from tests to review: the Burler review checks them instead.
3. Keep the doc rule, but drop the per-card test requirement for stencil-only changes.

## First step

Count, over the landed tasks of the last weeks, how much of each diff is tests that pin stencil or template wording, against the code change itself, so the choice rests on numbers.

## Result (count, 2026-10-15)

A Sonnet agent counted the 267 task landings on `main` since 2026-09-29 from `git log -p`, classing every changed line as stencil, wording test, other test, code, docs or other; its awk scripts and per-commit table are in `.scratch/test-weight/` in the prime, not committed.
Wording tests were: every `_test.go` under `contracts/stencils/`, every `*template*_test.go` and `*prompt*_test.go`, and in other tests the lines with `must:`/`mustNot:`/`want(All|None|Prompt|NotPrompt)` or a `Contains` call naming a prompt, stencil, template or render.

- Overall, wording tests are about 4% of code plus stencil lines (7k against 181k code and 5k stencil lines, three mass test-prune commits left out); the median commit has none.
- The weight sits in prompt-edit tasks with little code: in the 115 commits that changed a stencil, wording-test lines match or exceed stencil lines in 52, with a median of 1.2 wording-test lines per stencil line.
  Examples: `793c6a8ee` 20 stencil, 73 wording-test, 0 code; `1e39afc10` (`webster-pack-last`) 41 stencil, 96 wording-test, 91 code; `da58615cd` 54 stencil, 204 wording-test, 137 code.
- In tasks that also change much code the share falls to 5-20%.
- Uncertain: the regex over-counts a few lines that only mention a prompt and misses assertions in other idioms; the `contracts/stencils` test files also hold marker and registry tests, counted as wording; renames count as full delete plus add.

## Operator decision (2026-10-15)

Build option 1: stencil tests pin only what must hold (markers, file paths, order, and the presence of load-bearing rules), not the wording of sentences.

## Operator view, not a decision (2026-10-15)

It is odd to have mechanical tests for stencils at all.

## Open for the Discussion

- `contracts/stencils/claims_test.go` marks a claim that cites a provenance (crucible round, finding, sabotage proof, live observation) as protected and never dropped; whether such claims stay, shrink to a marker, or go is open.
- The orch's reading, not discussed: the PATTERN scans over stencils (`PATTERN-parent-directive`, `PATTERN-edit-directive`, `PATTERN-no-denied-recovery`) check structure and rules, not prose, and stay; marker sets, Fill round-trips and the registry tests stay.
- Where the rule that asks for a test per stencil change is written (stencils, PATTERN, CLAUDE.md), so it changes with the tests.
