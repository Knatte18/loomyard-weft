# Weaver: cold-start implementer with model escalation

Adds Weaver, a second implementer for tapestry's implementer row.
Weaver builds a plan batch by batch in one Go loop, with a fresh strand per attempt that climbs a model ladder on failure.
Only the batch agents are LLMs on the happy path; a stuck batch goes to shed's stop chain (rule table, tackler, arbiter, parent).
The default stays `webster`; a run chooses Weaver with `tapestry.implementer: weaver`, hub-wide or per task through `run_settings`.

## What changes

- `internal/weaverengine` (new) holds the row producer, state, journal, run directory and reset.
  Its package doc states the loop, ladder, journal and run directory.
- `internal/weavercli` (new) adds `lym weaver reset`.
- `internal/shedrecipe`: the closed `implementers` table gains `weaver`, built with the same edges, inputs, outcomes and `gates` key as Webster's row.
  No recipe row or routing edge changes.
- `weaver.yaml` (new config module, registered in `configreg`) holds `ladder` (default `sonnet[medium]`, `opus[medium]`, `opus[high]`),
  `fixer_timeout_min` (default 60) and `verify_gate` (default `webster`).
  An empty ladder or an unresolvable spec refuses at row build, naming `lym config weaver`.
- Weaver's stencils live in `contracts/stencils/weaver`.
- `_lym/weaver/` (state, reports, `journal.md`, `summary.md`, `outcome.yaml`) and `.lym/weaver/` (locks, prompts) are named through `lymdirs`.
- `internal/runplan` now holds the batch machinery both implementers share:
  run-plan state, report contract with a `journal` field, post-batch and suspect checks, failure digest, card gate rendering, strand await and verify-failure record.
  Webster's behaviour is unchanged; its `State` embeds the shared run-plan state beside its Master fields.
- Loom's readers of implementer state go through one active-implementer seam:
  the D1 guard, plan gate and its frozen-card note, renumber verbs, decision-notice guard, Describe, the PR rework archive and `lym fabric merge-in`'s in-flight warning.
- The plan-conflict record and route claim move to one implementer-neutral path, named through `lymdirs`; the tapestry Plan-Review text names it.
- `contracts/specs/implementer-spec.md` lists `weaver`, allows an implementer to commit its run directory and the plan directory during the run, and names the stop chain.
- Batten's watch reads `fixer_timeout_min` for a Weaver run's verify-fixer strand.

## Design decisions

- **Batches form one at a time.**
  Before each batch Weaver drops done cards (matched by slug) and partitions the rest with the batcher profile, recording the full partition.
  A reported rename therefore reaches the next partition, and the shared checks see pending cards, so only the true last batch reads as last.
- **Ladder.**
  Each failed attempt moves to the next rung; a machinery loss (lost strand, no report) gets one same-rung retry.
  A `NEEDS_PARENT` report stops at once.
  Every attempt that ends undone has its outcome recorded in state, then its strand stopped; a failed stop returns Stuck naming `lym reed remove <guid>`.
- **Re-entry.**
  After a stop the row re-reads the plan, so edits to card text carry into the next attempt.
  A stopped batch that passes the post-batch checks is recorded done with no spawn, unless the row has `gates` entries.
  Otherwise it gets one last-rung attempt; a `NEEDS_PARENT` stop resumes at the rung that asked.
  A plan whose card set no longer matches the recorded partition refuses.
- **Crash and resume.**
  The producer holds a run lock and fixes each attempt's report path before spawning.
  On a new call it stops strands whose outcome is recorded, then attaches through shuttle's `AttachGated` to attempts in flight.
  A report is recorded only from the attach's result.
- **Journal.**
  Each report's `journal` field is appended to `journal.md` once, keyed by heading, and later prompts and the arbiter's description name it.
  Weaver commits `_lym/weaver/` with the plan directory after each batch, never pushes, and the run's own push carries those commits.
- **Verify.**
  The plan-level verify runs through `verifytree.Verify` with one flakiness rerun.
  A failure starts one `gatefixer` fixer on the ladder, and the verify reruns after every fixer.
  `summary.md` carries `## Integration suite triage` and `## Verify gate fixes`.
- **Fixer and row gates.**
  The fixer carries no shuttle gates, so the row's `gates` entries ride the batch strands only and do not run over a fixer's commit.
  The shipped recipe's one entry, drift, is repeated downstream by Drift-Verify.
- **Guard.**
  The row refuses when the other implementer has begun batches in the current generation, and a corrupt state file refuses rather than reading as missing.
  Both refusals name their way forward.
- **`lym weaver reset`.**
  It moves the task branch to the run's start commit and restores the plan directory in the weft to the content at the run's start.
  It renames the run record under UTC-stamped names that no reader reads, and commits both through fabric.
  It refuses while the run lock is held or a probe by output file finds a strand; `--stop-strand` stops those strands first.

## To check by hand

Findings left open by review, each to confirm against the code:

- A same-rung retry allowance could span ladder cycles: confirm lost attempts and fixers are counted from the latest re-entry marker in `internal/weaverengine/run.go` and `resume.go`.
- A stale `outcome.yaml` or `summary.md` could survive a re-step: confirm they are removed at `Run` entry.
- A stopped batch's gate calls could go unrendered: confirm the stopped-batch check and the batch strand share one rendering.
- A failed strand stop for a `NEEDS_PARENT` attempt could record the batch kind in `internal/weaverengine/resume.go`.
- A rename reported through Weaver is not covered by a test.
- `lym weaver reset` in `internal/weaverengine/reset.go` could run its foreign-path check before the strand probe.
- A state-write failure after an attempt's outcome is not covered by a test.
- Rendering of the run's true last batch is not covered by a test.
- Not decided: whether a Done run with a row `gates` entry failing at HEAD after a verify fixer's commit is acceptable.

No integration suite failure, triage or fix is recorded for this run.
