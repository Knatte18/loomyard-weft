# Decision record — codex-astra-fan-bench

## Goal

Measure whether a Codex `gpt-6-astra` lens fan, three lens reviews and one handler review, finds what Claude's lens fan found on the `machine-handover` plan (`P4`), where solo Astra found none of its `BLOCKING` findings.
The result is a report with its data, graded against the Claude fan reference the earlier benches recorded.

## Scope

In:

- One input, `P4`, rebuilt from the solo bench's `data/inputs.txt` weft and code commits.
- One model and effort, `gpt-6-astra` at medium effort, one run per call, four `codex exec` calls in all.
- Three lens runs (`affected-set`, `guard-reach`, `way-forward`, the lenses of Claude's round 1) over a fresh snapshot copy each, then one handler run over the three lens outputs.
- Per call: wall time, fresh input tokens, the 5-hour and weekly window readings.
- Grading of the handler's findings, and of every lens finding the handler dropped, into `data/match.tsv`.
- `docs/research/codex-astra-fan-bench-<date>/report.md` with its `data/`, committed on the task branch.

Out:

- Any Claude run, any repeat run, any second input, any other model or effort.
- Any change to production code, stencils or PATTERN.

## Decisions

### D1. The harness is the solo bench's, adapted

The new `data/harness/` copies the solo bench's scripts, re-pointed at this worktree and its weft; `build.sh` takes the code commit from `data/inputs.txt` instead of a `--before` lookup.
The `P4` snapshot, decision record, plan, era stencils and focus directive are built as the solo bench built them.
Rationale: the same inputs and runner keep the fan row comparable to the solo rows.

### D2. A lens run is the round-1 explore step plus the fork rules, minus inherited context

A Claude fork inherits the handler's reading; a separate `codex exec` has none.
Each lens prompt is the round's explore step with its fork-reading block (the pre-read the forks inherited), then the fork boilerplate and `Your lens: <name>` with the lens text from `internal/lenses/template.yaml` at the code commit.
The boilerplate's inherited-context sentence and its six-tool-call cap are replaced by an instruction to do the pre-read first and then work the lens; the read-only rule and the final-message output rule stay.
Rationale: without the pre-read a lens run would review blind, and the cap exists only to stop a fork re-reading what it inherited.
The change favours Astra slightly on reading depth; the report states it.

### D3. The handler prompt is the round-1 handler with lens outputs in place of fork results

The handler prompt is the reviewer template with the fan cluster rules, minus the spawn instruction: the three lens final messages are handed over as files, and the handler is told the forks have returned.
Its holistic review, consolidation, `origin:` tags and `## Rejected` section stay as the stencil has them.
Rationale: the handler's judgement over lens findings is what the fan is.

### D4. Window guard

The 5-hour window is read from the newest rollout before the first call and after each call.
The bench stops, and the report says what was done, if a reading passes 70 percent.
Rationale: the task's cap.

### D5. Grading

Findings are graded `shared`, `unique-real` or `false` against the `P4` fan findings and the snapshot, as the solo bench graded, by this session.
A lens finding the handler rejected is graded too and marked dropped.
Rationale: the task asks for dropped lens findings to be graded; one grader is a stated limit.

## Constraints

- `PATTERN-lym-temp-root` and the repository's scratch rule: bench work lives under `.scratch/`, which is not committed.
- Agents run in interactive tmux never `claude -p`; the bench runs `codex exec`, as the solo bench did, and no Claude call is made.
- Markdown with semantic line breaks.
- `codex exec` runs against a scratch `CODEX_HOME` linked to the operator's `auth.json`.

## Auto-mode assumptions

- The date in the output directory name is the date the bench runs.
- `PATTERN-test-speed` and the other PATTERN entries concern code; the change adds no code beyond bench scripts under `data/harness/`.

## Open risks

- One run per cell, one grader: no variance is measured.
- The lens runs read for themselves, so the fan's cost is not the cost of forks that inherit context.
- The 5-hour window may pass 70 percent before all four calls run.

## Acceptance criteria

- Four `codex exec` calls ran, or the report names the stop and what ran.
- `report.md` carries the summary row beside the solo Astra and Claude fan rows: findings, fan findings found, `BLOCKING` found, verdict, wall time, tokens and window points.
- `data/match.tsv` grades every handler finding and every dropped lens finding.
- `data/` holds the harness, the prompts, the lens outputs, the handler review and the usage line per call.
