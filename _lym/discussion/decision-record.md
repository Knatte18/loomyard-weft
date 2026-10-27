## Goal

Measure, on plan `P4` (`machine-handover`), whether block v2 with a fork cap of 20 tool calls lets Codex Astra's forked lens fan find the Claude fan's findings the earlier runs missed, and whether one high-effort run or the union of two medium-effort runs is the better buy.
The output is a research report with its data, not product code.

## Scope

- Three forked fan runs of `gpt-6-astra` on `P4`, each one `codex exec` handler spawning three lens forks with `fork_turns="all"`: one at `high` effort and two at `medium`, identical but for effort.
- The tuned-prompt bench's harness, rebuilt under this task's `data/harness/` with its path root changed to this worktree.
- Block v2 from the board entry, word for word, in `data/prompts/block-v2.md`, in the handler prompt and in the fork boilerplate.
- The fork boilerplate's cap sentence changes from `at most 6 tool calls` to `at most 20 tool calls`; nothing else in the boilerplate changes.
- Grading of every handler finding and every dropped fork finding against the Claude fan reference of `P4`, into `data/match.tsv`.
- Output `docs/research/codex-v2-p4-bench-<date>/report.md` with its `data/`, landed as the run's commit.
- Out of scope: any Claude run, any other repeat, any effort above `high`, any change to `lym` code, PATTERN or the board.

## Decisions

### D1: Reuse the tuned-prompt bench's harness unchanged but for paths, effort, block and cap

Rationale: every condition but the block, the cap and the effort must match the earlier `P4` rows for the comparison to mean anything.
The inputs are the weft commit and code commit already recorded in the earlier bench's `inputs.txt`.

### D2: Run the two medium runs and the high run as three separate cells with separate scratch `CODEX_HOME` sessions

Rationale: each cell copies its own working tree and collects only its own rollouts, so a run cannot read another run's review file.
The medium runs count as run in parallel for the wall-time comparison, since the cells share no state.

### D3: Stop at 70 percent of a rate-limit window

Rationale: the board entry sets the stop.
The window is read from the last `token_count` of each rollout before the next run starts.
A stop reports what was done, with the figures of the completed runs.

### D4: Grade by the earlier benches' rule

Rationale: `shared` names the same defect as a fan finding, `unique-real` holds on reading with no fan finding of it, `false` does not hold.
The fan's finding texts are not in the repository, so a match rests on the earlier benches' `match.tsv` notes and severities, and the report says so.

### D5: Report the medium pair's agreement as the first repeat measure

Rationale: the two medium runs differ only by run-to-run variation, so their overlap and their difference in found fan findings is the bench's variance reading.
The report states it as one pair on one plan.

## Constraints

- `PATTERN-lym-temp-root` and the worktree rule: scratch files go under `.scratch/` in this worktree, removed by literal path; the bench stays out of git except `docs/research/codex-v2-p4-bench-<date>/`.
- Codex runs are the research subject and are `codex exec`, not agents lym spawns, so the interactive-tmux rule does not apply to them.
- The scratch `CODEX_HOME` holds only a link to the operator's `auth.json`.
- Markdown follows semantic line breaks; table cells stay on one line.
- No effort at `xhigh` or above.

## Auto-mode assumptions

- The Codex CLI, its sign-in and the weft and code commits of the earlier benches are available on this machine.
- Sibling benches (`codex-v2-p1-bench`, `codex-v2-p2-bench`) run on the same account at the same time; the report says so beside every window figure.

## Open risks

- One input, one run per cell and one grader: no setting's count is separated from run-to-run variation, and the medium pair is the only repeat.
- The grader is a Claude session and may favour its own family's phrasing.
- Concurrent sibling runs may push a window past 70 percent before a run starts; the report records where the stop fell.
- The fork prompts are encrypted in the rollouts, so that a fork saw block v2 rests on the handler following the cluster rules and on the forks' citations.

## Acceptance criteria

- `report.md` holds a comparison of the high run, each medium run alone and the union of the two medium runs, each with findings, Claude fan findings found, `BLOCKING` found, unique real, false, fresh input tokens, window points and wall time.
- The report says, for each of the Claude fan's three `BLOCKING` findings on `P4`, which run found it.
- The report states how far the two medium runs differ, and carries every earlier Astra and Claude `P4` row beside the new rows.
- Rollout evidence for each run shows the forks started, ran concurrently, inherited the handler's context and loaded `AGENTS.md`; a run failing that is reported and not graded.
- `data/` holds `block-v2.md`, the harness, the prompts, the reviews, `match.tsv`, `usage.txt`, `rollout-evidence.txt` and `context-hashes.txt`.
- Per run, the report records wall time, the handler's and each fork's fresh input, output and reasoning tokens, each fork's command count and the window readings.
