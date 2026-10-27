# Orch note

Runs, the board and anything else `lym` can read stay out of this note: the next session reads them from `lym`.

The operator chats in Norwegian; reply in Norwegian, short and plain (they follow from the phone); write every file, board entry and agent message in English.
The operator does not remember earlier context: when asking for a decision, present the whole case (problem, numbers, options weighed, proposal) every time, never only a delta, and never refer to proposals by bare letters without restating them. Explain plainly: give the simple version first, and word things precisely (say "recipe", not "profile", for loom/darn/pawl). Check numbers before quoting them.
The operator wants every rule in the repo's instructions (the orch role stencil, the orch rules files, CLAUDE.md for what every agent needs), never in file memory. Orch-only rules live in the hub-wide `_board/_lym/orch/rules.md` (written with `lym orch rules --from <file>`; a draft starts as a copy of the hub file) plus each orch's own weft `_lym/orch/rules.md`.
While the operator discusses a design, discuss first and write it to the board only after they agree. Investigate yourself with filtered greps and short reads. Explorations run as darn docs tasks with an exact run cap (hub rule).
Never `cd` into another worktree in Bash (it moves the session's working directory); use `env -C` or absolute paths, and pass `--body-file` an absolute path.
Tasks: release only on the operator's explicit go-ahead naming that step (hub rule). Always set `depends_on` and module labels on every new task, and set `depends_on` between tasks whose packages overlap. No run or subagent may file a task, only notes.
Decisions for a run: `lym loom decision add <slug> --by parent` via `env -C <worktree>`; `--amends-plan` only once a plan is approved. Technical calls a fixer hands me I decide myself after a quick check, then tell the operator; design questions go to the operator.
Recipe choice: single bug notes run darn when the operator names darn; never propose one note alone as loom. Profiles: speed/light/solid/wide plus docs; defaults darn=light, loom=solid.
Never say "dere". Say "worktree", never "pair". A slug must say what the entry does. Never record an operator remark as a decision. Never set a priority unless the operator says so. Handles get reused; re-read them.
Design principles the operator stated: fabric knows nothing about IDEs or shuttle; side actions are ordinary shed producers with a config flag; anything every FSM needs belongs in shed; lyx hard-codes no language, engine or branch name; bench before build; the prime is the root beam; a beam is a worktree, local to one machine; ownership sits on root and beam projects; batten stays identical for every recipe; a perch is expensive, so avoid re-running one (E11 `loom-finalize-merges-itself`).
The operator is convinced a lens fan reviews far better than a solo reviewer (`docs/research/codex-astra-solo-bench-2026-10-25/report.md`).
Codex: ChatGPT Plus, testing and benching only.
Machines: Neve (this), OSL-1046 (Ubuntu 24.04, has Models), OSL-1033 (Windows, out of focus). Quarry and C# sandbox (`cs`) hubs have their own orchs.
Never run `lym config <module>` without `--set`, `--print` or `--resolved`. Never run an upsert and an Edit/Write of its body file in the same parallel batch.
There is no single verb that pulls the whole hub while the orch runs: `lyx fabric pull` covers the prime's own pair only, not Bolt (`_board`); the hub-scope origin reconcile runs inside `beam start`, `adopt`, `cleanup --apply` and the hibernate wake. A note for a standalone verb was offered to the operator, not filed.

## Doing now

This note was written in the old hub `loomyard-LYXHUB` on 2026-10-27 for the move to `lym`: everything untracked (`.lyx/`, `.scratch/`) stays behind, so this note is committed on `main-weft` as `_lyx/orch/handoff-lym-cutover.md` and reaches the new hub as `_lym/orch/handoff-lym-cutover.md`.
The new hub's orch reads it from there, then it is removed from `main-weft` in a weft commit.

- T1 `rename-binary-lym` (darn) renames lyx to lym; it was at Darn-Write verify when this note was written. Its decisions so far: remove the empty untracked `cmd/lyx` directory so the old gate's prebuild skips; keep the comment reflow (`tools/godocreflow`) in the diff (operator).
- The landing deploys nothing: `deploy_branches` in the landing config is `[]`.
- After the landing the operator runs the cutover in E4 `lym-cutover-steps` (stop all, deploy from the prime, `git mv _lyx _lym` on Bolt and `main-weft`, fresh `lym fabric clone`, then `deploy_branches` back to `["main"]`, repoint the `loomyard` plugin marketplace path in `~/.claude/settings.json` and `~/.claude/plugins/known_marketplaces.json`, `update-plugins.sh` from the new prime, `lym beam start`).
- Root is owned by Neve again (`lyx adopt root`, 2026-10-27); `max_runs` is 5 in the prime's layer.
- A test of the darn run's gate wrote a trace into the real hub at `_board/.lym/logs/` (untracked): a test reached the live hub's board dir. Not yet raised with the operator or filed.

## Next step with the operator

- Once in the new hub: check that the board, the orch rules and the config read under `_lym/`, and that dobby is up with `max_runs` 5.
- The quarry, cs and OSL-1046 hubs take the same cutover (steps 1, 3, 4, 5 of E4).

## Waiting on the operator

- Whether to file a bug note: an instruction the operator types into a worker's pane is lost when the arbiter (tackler) re-spawns the row, and the old darn asked the orch the same question the tackler settled at the same moment.
- Whether to file a note for a verb that pulls the whole hub (Bolt included) while the orch runs.
- Codex fan bench proposal (a darn docs task, one run: Astra medium with 3 lens forks on one plan) is not on the board; no answer yet.
- E8 `darn-drift-await-row` and E11 `loom-finalize-merges-itself` stay notes until the operator promotes them.
