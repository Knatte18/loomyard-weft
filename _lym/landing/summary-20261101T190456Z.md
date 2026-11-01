# Split the plan spec by need and cut it to half its size

`contracts/specs/loom-plan-spec.md` was 112 KB, about 28k tokens.
No agent could read it whole, and half of it was the validation-check catalogue.
This change cuts the spec down and splits it so each role reads only the part it uses.

## The new files

| File | Holds | Read by |
|---|---|---|
| `loom-plan-spec.md` (24 KB) | the card format: layout, fields, types, ref shapes, `root:`/`//`, rename and move, `plan:` handles, numbering, the Pattern-Departure trailer, the verify model, granularity and a worked example | the Plan writer, Plan-Review, Code-Review, Webster's implementer and fixers |
| `loom-plan-checks-spec.md` (19 KB, new) | every plan-gate check: what raises it and its way forward, grouped by topic, plus where each entry point runs | a reader whose `lym loom validate-plan` finding does not settle the matter, Plan-Review, the plan fixer |
| `loom-plan-csharp-spec.md` (5 KB, new) | the C# glyph alphabet, C# handles, re-sign bounds, the C#-only checks and the checks a `csharp` plan skips | a writer whose plan sets `language: csharp` |
| `loom-parts-spec.md` (gains a `## Part plans` section) | the part scope and the join of the parts' plans, moved from the plan spec | part writers and the chair, which already read this file |

Together the plan files are 57 KB, against 112 KB before.

## What was cut

- History and superseded design: "Batch is gone", the design doc's table versus its prose, the deferred concurrency and forward-compat sections, and the quarry degraded-mode section.
- Rationale no reader acts on, such as the why behind each hard rule and the handle-collision story.
- Detail that restates code. Examples are the check-order exceptions, which now point at `validate.go`'s file comment, the caller-uncovered scan's import-binding rules and the fabric-vocabulary name rules.
- Long "Bound:" and "it only refuses, it misses…" paragraphs. The gate's own finding carries the way forward.

The worked example no longer shows the retired `## verify:` section.
Its card 1 now declares `RowJSON` as a `plan:` handle, and its `Verify:` line is a `lym gate test --wait` call, so the example obeys the rules the spec states.

## References that follow the new files

- `contracts/specs/specs.go` embeds and registers `loom-plan-checks-spec` and `loom-plan-csharp-spec`, so `stencilstore` deploys them under `specs/loom/` beside the plan spec. `specs_test.go` pins the new names and their paths.
- The plan-format blocks and the Plan-Review rubric now cite the checks spec wherever they name a check or the check set, and the overview block cites the C# spec for the C# rules.
- The cold plan fixer's prompt points at the checks spec for the checks its findings name.
- Go comments that cited the old "Validation checks" section now cite the checks spec. These are in `planparser/validate.go`, `planglyph/doc.go`, `webstercli validate --help` and `planparser/validate_test.go`.
- `docs/overview.md` lists the new spec files.

## Design decisions

- `loom-plan-spec.md` stays the entry point under its old name, so the many existing readers need no change: the implementer, the code-review rubric, the PATTERN directives, the tapestry recipe and the Pattern-Departure citations. It opens with a list of the contract's files and says when to read each one.
- The part scope and the join moved into `loom-parts-spec.md` rather than a new file, because the part writers and the chair already read that spec.
