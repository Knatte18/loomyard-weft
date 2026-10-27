## Goal

Answer on plan `P2` (weft commit `394715f7f`, code commit `750903aef`) whether block v2 with a 20-call fork cap lets the forked Codex Astra lens fan find more of the Claude fan's 13 findings and 3 `BLOCKING` ones than block v1 did, and whether one high-effort run or the union of two medium-effort runs is the better buy.
The deliverable is `docs/research/codex-v2-p2-bench-2026-10-27/report.md` with its `data/`, landed as the run's commit.

## Scope

- One input, `P2`, rebuilt from the earlier benches' commits in `docs/research/codex-review-bench-2026-10-25/data/inputs.txt`, with the Claude fan's three round-1 lenses `affected-set`, `guard-reach` and `way-forward`.
- Three forked fan runs of `gpt-6-astra`, each one `codex exec` handler spawning three lens forks with `fork_turns="all"`: one at `high`, two at `medium`, identical but for effort.
- The harness is the tuned-prompt bench's `data/harness/`, copied into the new `data/harness/` with its root path changed and parameterised by effort and cell; its context is the inlined PATTERN overview and the snapshot's `CLAUDE.md` as `AGENTS.md`.
- Block v1 is replaced by block v2, word for word, in the handler prompt and the fork boilerplate; the boilerplate's cap sentence changes from `at most 6 tool calls` to `at most 20 tool calls` and nothing else in it changes.
- Hand grading of every handler finding and every fork finding a handler dropped, in `data/match.tsv`, against the Claude fan reference of `P2`.
- No code under `internal/` or `cmd/` changes, no Claude run, no other repeat, no `xhigh` effort.

## Decisions

### Decision: the bench runs from this worktree's `.scratch/`

Decision: harness paths use this worktree as `ROOT`, and the snapshot comes from `git archive` of code commit `750903aef`, which this repository holds; `.scratch/` is not committed.
Rationale: the earlier harness names a sibling checkout that does not exist here; the worktree holds the commit and the project rule puts scratch files under `.scratch/`.

### Decision: one scratch `CODEX_HOME` per run

Decision: each of the three runs gets its own `CODEX_HOME` holding a link to the operator's `auth.json`, and copies only its own rollouts.
Rationale: the runs overlap in time, so a shared sessions directory would give each cell the others' rollouts under the runner's `-newer` copy.

### Decision: the three runs start together

Decision: the high run and the two medium runs start at the same time, after a window check, so the two mediums count as run in parallel as the task states.
Bound: the window is read before the start and from each rollout's last `token_count` after it; a reading past 70 percent ends the bench with what was done, and a run that fails its fork evidence check is reported, not retried.
Rationale: parallel mediums are the premise of the union row's wall time; the sibling benches share the account, so the window points cannot be attributed to one run and the report says so.

### Decision: the cap sentence is edited in `build-forkfan.sh`

Decision: the harness changes the cap sentence's number in the cluster rules the build produces, and checks that `at most 20 tool calls` is present and `at most 6 tool calls` absent.
Rationale: the task allows exactly that one change to the boilerplate; a check turns a missed anchor into a build failure.

### Decision: grading follows the earlier benches

Decision: each handler finding and each dropped fork finding is `shared` with a Claude fan finding, `unique-real` or `false`, decided by one reading of the cited card lines and snapshot code; the union row counts distinct findings found by either medium run.
Rationale: the same labels keep the rows comparable with the earlier `P2` rows.

## Constraints

- `CLAUDE.md`: agents run in interactive tmux, never `claude -p`; the bench's `codex exec` is the Codex CLI the task prescribes and not a Claude agent.
- `PATTERN-test-speed` and the gate: `docs/` text needs no test run; the change touches no Go code.
- Markdown follows semantic line breaks, one sentence per line, table cells on one line.
- Scratch files stay under `.scratch/` and are removed by literal path.
- Stop and report what was done if a rate-limit window passes 70 percent.
- Stop and report if a run's rollouts do not show the forks started, ran concurrently and inherited the handler's context, and that `AGENTS.md` loaded.

## Auto-mode assumptions

- The Codex CLI is installed and signed in on this machine, as the earlier benches found; a missing sign-in stops the bench with a report.
- `gpt-6-astra` is available on the account at `high` and `medium`.
- The sibling benches `codex-v2-p1-bench` and `codex-v2-p4-bench` run on the same account at the same time.

## Open risks

- One input and one run per setting: a difference of one or two findings can be run-to-run noise, which the two medium runs measure for the first time.
- The grader is a Claude session and may favour its own family's phrasing.
- Fork prompts are encrypted in the rollouts, so that the forks held block v2 and the 20-call cap rests on the build check and the forks' behaviour.
- A rate-limit window shared with three sibling runs can pass 70 percent before the bench ends.

## Acceptance criteria

- `report.md` states per run and per setting (high, each medium alone, union of the mediums): findings, Claude fan findings found, `BLOCKING` found, unique real, false, fresh input tokens, window points and wall time.
- The report names, for each of the Claude fan's three `BLOCKING` findings on `P2`, which run found it.
- The report states how far the two medium runs differ in findings, in fan findings found and in tokens.
- The report puts every earlier Astra and Claude row for `P2` beside the new rows.
- Per run, the report records the handler's and each fork's fresh input, output and reasoning tokens, each fork's own command count, and the window readings, with the sibling-bench caveat beside the window figures.
- The rollout evidence of each run shows the forks started, ran concurrently, inherited the handler's context, and `AGENTS.md` loaded.
- `data/` holds `prompts/block-v2.md` equal word for word to the block in the board entry, the harness, the reviews, `match.tsv`, `usage.txt`, `rollout-evidence.txt` and `context-hashes.txt`.
