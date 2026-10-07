# Burler: split a round into a reviewer and a fixer, model per round

A Burler round now runs two agents that start together: a reviewer that writes the review only, and a fixer that orients itself while the review runs, then validates and fixes the findings.
`loom.yaml` picks each half's model per round, so a cheap reviewer and a stronger fixer can differ, and a fresh base model can review after round 1.
Before this, one agent did both jobs under the single `review` model-spec, so the halves could not be set apart and the fixer's orientation never overlapped the review.

## What changes

- `burlerengine`: `Engine.Run` starts a reviewer (role `burler-review`) and a fixer (role `burler-fix`) and returns one `Result` for the round.
  `Result` carries each half's session id, strand guid, last message and run directory.
  The engine is told a `StrandRemover` (`NewReedStrandRemover`), which is how it stops a half.
- Handoff: Go writes a ready marker once the reviewer's run is done, its review parses strictly and, for a cluster round, the fork audit passed.
  The fixer runs the new read-only verb `lyx burler await-review <marker-path>`, which polls for that file under a cap and reports `ready` true or false.
  `burlermarker.Path` is the one derivation of the marker path from the review path, shared by `BurlerProducer` and `lyx burler run`.
- Failure: if either half fails, the round fails with that half's outcome and the other half's strand is removed.
  A fixer that reaches done before the marker was written ends the round with an error naming the skipped handoff.
  A fixer that changes the review file after the handoff ends the round with an error naming the file and no verdict.
- Fixer report: the fixer may list a finding in a new `Disputed` section, with evidence, only when it shows the finding's premise false.
  Every other finding is fixed.
  The next round's prior-rounds block names disputed findings beside deferred ones.
  `pattern/PATTERN-review-round.md` and `PATTERN.md` are updated to match.
- Models: `loom.yaml` gains a `fix` key beside `review`.
  Each takes one model-spec or a list; entry *n* applies to round *n* and the last entry to later rounds.
  An existing scalar `review` loads unchanged, and an empty list or bad entry fails config load.
  Template defaults are `review: sonnet[medium]` and `fix: opus[medium]`.
  `loomengine.ResolveReview`, `shedrecipe.Env` and `shedadapters.BurlerProducer` carry both lists.
- Recipe: the `BurlerRound` row loses its `model` and `effort` keys, and `configRejectUnknown` now rejects them.
- CLI: `lyx burler run` replaces `--model` and `--effort` with `--review-model`, `--review-effort`, `--fix-model` and `--fix-effort`.
- Stencils under `contracts/stencils/burler/`: a reviewer orchestrator and a fixer orchestrator replace the single round orchestrator, and the step files go to the half that runs them.
- Shuttle: `Runner.ProbeGated` returns the attach probe's handle without waiting; this is the only shuttle addition.
- Resume: `ProbeRound` and `Resume` classify each half by its own output file.
  With both halves live, `BurlerProducer` reattaches to both with the waits concurrent.
  With one half live or missing, it stops the live half and re-runs the round from attempt 1.
  No resumed round ends with a live half or spawns a second agent for one.

## Design decisions to check

- The two-agent round stays inside one producer, so Shed still sees one `BurlerRound` step with one outcome.
  A generic two-agent producer (`two-agent-producer`) and agent messaging (`creel`) are out of scope.
- The fixer's write surface lists the fixer-report only, in every fix scope, and the caller's gate entries such as `verify` run on the fixer's run only.
  The review-parse gate belongs to the reviewer's run.
- `review_timeout_min` is the deadline of each half, counted from that half's own spawn.
- Not mechanically enforced: the fixer's read-only discipline before the marker, and its refusal to create the marker or hand the wait verb another file.
  The stencil states both rules, and review is the only catch.
  Only the review file is compared after the fixer.
- The reviewer's fan: `burlerengine` keeps the cluster flow wherever a profile names a fan, and no recipe row names one.
- Breaking change: scripts that pass `--model` or `--effort` to `lyx burler run` now fail.
  Only in-repo callers and tests are known.

## Not run by hand

- The rewritten tmux smoke test `internal/loomcli/smoke_attachprobe_test.go` was reviewed but not run.
- Run a real `lyx burler run` with both halves live, to see the fixer wait on `await-review` and then fix.
- Run a loom round on a resumed run with both halves live, to check the reattach.
