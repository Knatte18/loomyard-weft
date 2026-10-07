# Round 4 fixer report

Every finding in `round-4-review.md` is fixed; `lyx loom validate-plan` exits clean (informational findings only) over the edited plan.
`approved:` in `00-overview.md` is untouched (`false`).
The round-1 to round-3 fixes are all still in place, and none of those rounds deferred anything.

## Changes

- **F1 (MEDIUM, scope: plain start errors)** — `07-two-agent-round.md`'s `Run` paragraph now covers a plain `StartGated` error (spec validation such as a pre-existing fixer-report, run-directory creation, `AddStrand`, run-state persistence):
  the reviewer's is returned with the fixer never started, and the fixer's stops the already-started reviewer first, or is returned wrapped as `ErrHalfNotStopped` when that stop fails.
  The failure-rules test gains rows for a plain reviewer start error, a plain fixer start error (reviewer guid removed), and a failing stop.
- **F2 (MEDIUM, scope: `cli.go` target)** — `internal/burlercli/cli.go` is now an Edit target of card 7, and the `burlercli` paragraph says `burlerCLI` gains `markerRoot` and `markerBase` fields, set by `wireHub` and `wireStandalone`.
- **F3 (LOW, consistency: pathRules row inputs)** — `02-ready-marker-path.md` now has the row pass a review path relative to the anchor, with `l.AnchorPath()` as root and base, and says why a path built from the (relative) anchor would be resolved twice.
- **F4 (LOW, consistency: ImpactSummary exclusivity)** — card 1's summary names the attach test and shuttle doc beside the budget row; card 7's drops the "only other files" claim and states the deleted stencil's blast radius (every reference is a target of the card or of card 5).
  Card 5's summary now also names the pattern docs and the directive stencils.
- **F5 (LOW, consistency: test economy)** — `08-two-half-resume.md`'s `Resume` test keeps the resume-only behaviour and one failure row proving the attached handles route into `join`; the other failure outcomes stay pinned once, in card 7's tables.
- **F6 (LOW, scope: directive stencils)** — card 5 adds `pattern-directive-review-fix.md` and `friction-directive-review-fix.md` as Edit targets (header comments, part A and part B wording, the fixer's own phases);
  card 7's `prompt.go` paragraph rewords `clusterRulesBlock` and `toolUseRules` and moves the `prompt_test.go` assertion on the old sentence.
- **F7 (LOW, consistency: marker I/O halts)** — card 7 places the stale-marker removal and marker create or write failures outside the refusal table beside the skipped-handoff and changed-review errors, with messages naming the marker path.

## Deferred

None.
