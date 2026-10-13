# Lighter test rules for small changes

`lighter-test-rules` · Note · enhancement, stencil

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
