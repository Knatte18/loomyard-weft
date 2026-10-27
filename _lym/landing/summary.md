# Bench a Codex Astra lens fan against the Claude fan on P4

Adds `docs/research/codex-astra-fan-bench-2026-10-27/`: a report, its data and its harness for a bench of `gpt-6-astra` at medium effort running the Claude fan's shape over the `machine-handover` plan (`P4`), as three lens runs and one handler run, four `codex exec` calls in all.
No production code, stencil or PATTERN entry changes.

## What the bench found

- The Astra fan found none of the Claude fan's 11 findings and none of its 3 `BLOCKING` ones, where solo Astra had found 2 of 11 and no `BLOCKING` one.
- The handler returned `BLOCKING`, as the fan did, with 8 findings that all hold on reading and are absent from the fan's list; solo Astra had 2 such findings.
- The fan took 907 s and 466k fresh input tokens over four calls, against 166 s and 77k for solo Astra, and at most 3 points of the weekly window; the ChatGPT Pro plan reports no 5-hour window.
- With one input, one run per call and one grader, the sample does not rank the fan against solo Astra or against the Claude fan.

`report.md` holds the summary table beside the solo Astra, solo Opus and Claude fan rows, the per-call figures, the handler's findings and the limits.
`data/match.tsv` grades every handler finding and the one lens finding the handler dropped.

## Design decisions

- The harness is the solo bench's, adapted: `build.sh` takes the code commit from `data/inputs.txt` rather than looking it up, and `build-fan.sh` and `run-call.sh` are new.
- A lens run is the round's explore step with its fork-reading block, then the fork rules and the lens text from the code commit's `internal/lenses/template.yaml`.
  A Claude fork inherits the handler's reading and a separate call cannot, so the fork boilerplate's inherited-context sentence became an instruction to read first and its six-tool-call cap was dropped; this favours Astra on reading depth and the report says so.
- The handler prompt is the round-1 reviewer with the fan cluster rules, its spawn paragraph replaced by the three lens outputs as files in its copy.
- The calls ran in sequence, so each window reading belongs to one call.
- The grading follows the solo bench's labels, with one grader, and covers the handler's findings and the dropped lens finding.

## Verification

The bench changes no code, so no test applies.
Every handler finding and the dropped lens finding was checked against the cited card lines and the snapshot's code (`cmd/lyx/main.go`, `internal/fabricengine/pull.go`, `internal/landingshed/finalize.go`, `internal/pauserecord/sleep.go`, `internal/fabricengine/failedpush.go`).
