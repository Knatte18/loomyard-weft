# P1 codexengine

Rendered 2026-11-17 11:23 UTC; every age below is as of then.
Under: [Board](README.md).

[`codexengine`](design-codexengine.md) · Age 21h · Owner `neve`

## Goal (operator, 2026-11-03)

`codexengine` landed with project `codex-engine` and has run motif runs well, but was never tested as a whole.
A stress test proves, on real launches, that the engine does everything its design says, the hooks above all, and records each fault as a bug note.

## Codex parity (from the Sketch `codex-same-as-claude`, 2026-11-16)

A Codex session lym gives a role gets exactly what a Claude session in that role gets.
The tasks raddle filed from that Sketch build it:
`prompt-engine-per-launch`, `codex-standing-instructions`, `codex-wording-engine-blocks`, `codex-question-deny-nudge` and `launch-manifest-parity`.
The two projects `codex-engine-trial` and `codex-parity` were merged into this one (operator, 2026-11-16).

## What the engine is designed to do

The package doc `internal/shuttleengine/codexengine/doc.go` is the spec; each part below tests one area of it against real Codex sessions launched through `lym shuttle`.

1. Home and trust: the shared lym Codex home under `CODEX_HOME`, `auth.json` and sessions linked to the operator's home, `config.toml` rendered whole under a file lock, trusted roots for a worktree and a linked worktree, dropped roots for a gone folder, the read-back Warn, and the folder-trust prompt backstop.
2. Sandbox and refusals: `-a never -s workspace-write`, the records checkout as a writable root, a `.codex/rules` folder on disk, tracked by git or not, refusing the launch and a `.codex/config.toml` on disk named in a Warn (the checks read the folders on disk since `codex-trial-1-fixes`).
3. Hooks: SessionStart, UserPromptSubmit, Stop with its reply, PermissionRequest and PostToolUse each reach the run's events file or activity stamp; an interrupt reads as interrupted through `turn_aborted`; the context size from `token_count`.
4. Shell guards: every installed guard row denies its command through the PreToolUse hook with its Codex steer, the prompt file carries the notices, and a hook that fails to run is seen failing open.
5. Pane and send: the composer, the busy line and the trust prompt read right; a long or multi-line send through the guarded paste-then-Enter lands whole; a dead pane after exit.
   Creel by the pane path, the only one a Codex strand has: a program's notice typed in full into an idle session, an agent's mail reminded with `run lym creel read`, and the Codex session running `lym creel read` and `lym creel post --reply` from inside its `workspace-write` sandbox, which must reach the creel daemon.
6. Skills: a `<plugin>:<name>` load reads the newest version's `SKILL.md` and the load is seen in the rollout; an unknown skill is reported unknown.
7. Stress: several Codex sessions at once, long sessions, repeated interrupts and sends, and a darn run with a Codex seat end to end.

## Rules

- Each part writes its findings to a report under `docs/research/`, and files each lym fault as a bug note on this board.
- Parts run as darn with profile `docs`, unless the operator says otherwise.
- A Codex run gets `high` effort at most.

## Built since (tidy, 2026-11-05)

Task `codex-creel-hooks` (`e3aedbc5`) gave Codex strands creel delivery through hooks, as Claude strands have: mail reaches a running session at the end of a tool call through `PostToolUse`, at a turn end through `Stop`, and an idle session through `codex queue` run by the strand's `hook-wait` waiter. So part 5's "the only one a Codex strand has" holds only for a strand opened without a prompt file (`OpenLaunch`, a standalone run), which stays a pane-mail strand.
Task `codex-creel-trial` (`afa2cf2e`, `docs/research/codex/engine-trial/codex-creel-trial-2026-11-05.md`) trialed it on real sessions in `lyt` and found one fault, filed as note `codex-idle-wake-extra-reminder`.

## Built since (tidy, 2026-11-07)

Task `codex-skill-load-false-missing` (`0e02aaad`) fixed a false `skill skipped ... cause="not loaded"` in part 6's load check.
`ClassifySkillLoad` now also reads a skill as loaded when a tool call output of the load turn holds the whole of its `SKILL.md`, so a seat that reads its skills through a path built at run time (a `cat .../skills/${name}/SKILL.md` loop) is no longer reported missing.
A partial output still leaves a skill missing.
The fix is checked against unit cases only: the original rollout and a live Codex run were not used, so part 6 should still see the load on a real session.

## Built since (tidy, 2026-11-08)

Task `codex-sandbox-mode` (`539525a0`) made part 2's sandbox a machine setting: `codex.yaml` has a `sandbox_mode` key, `workspace-write` by default or `danger-full-access`, set with `lym config codex --layer machine`.
A `danger-full-access` launch carries no sandbox flags, no network setting and no writable roots, and `OpenLaunch` does the same.
The change was checked by unit tests only.

## Built since (tidy, 2026-11-11)

Commit `9874c966` (task `codex-bubblewrap-sandbox`) added a third `sandbox_mode` value, `auto`, now the template default, so part 2 no longer needs the mode set by hand on a machine whose bubblewrap cannot start.
A launch under `auto` or `workspace-write` first probes the sandbox (`codex sandbox -c sandbox_mode="workspace-write" -- true` under the lym Codex home in the worktree):

- Under `auto`, a sandbox that starts runs `workspace-write`; one that refuses (OSL-1046's AppArmor-restricted user namespaces) runs `danger-full-access` with a warning naming bubblewrap's output.
- Under an explicit `workspace-write`, a sandbox that refuses refuses the launch, naming the cause and `lym config codex --layer machine --set sandbox_mode=auto`; the seat reads as not started, not stopped.
- `danger-full-access` runs no probe.

Both `Prepare` and `OpenLaunch` settle the mode through `launchSandboxMode` in `codexengine/sandbox.go`; `Prepare` refuses before `prompt.md` is written.
Part 2 therefore tests the probe, its three outcomes and the refusal on a real machine where bwrap fails (OSL-1046) and on one where it works (Neve), not only the fixed `workspace-write` flags.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (5)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T13 | **A Codex session gets Claude's standing items, user setup, instruction files and subagent terms in its developer instructions** | [`codex-standing-instructions`](design-codex-standing-instructions.md)<br>[Sketch](decision-codex-standing-instructions.md) | tapestry · solid | Plan-Burler-Fix | neve | 21h<br>21h | codex, shuttle, stencil, heddle |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (2)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T17 | **Claude-only wording moves into Claude engine blocks, each with a Codex copy, held by a term scan** | [`codex-wording-engine-blocks`](design-codex-wording-engine-blocks.md)<br>[Sketch](decision-codex-wording-engine-blocks.md) | darn · solid | T13 | 21h<br>21h | codex, shuttle, stencil, heddle |
| T18 | **A Codex session gets the question-tool deny and the batching nudge Claude runs** | [`codex-question-deny-nudge`](design-codex-question-deny-nudge.md)<br>[Sketch](decision-codex-question-deny-nudge.md) | darn · solid | T13 | 21h<br>21h | codex, shuttle, stencil, heddle |

### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T19 | **Each launch writes its manifest, a parity test compares the engines per seat, and lym shuttle manifest prints both** | [`launch-manifest-parity`](design-launch-manifest-parity.md)<br>[Sketch](decision-launch-manifest-parity.md) | tapestry · solid | T17<br>T18 | 21h<br>21h | codex, shuttle, stencil, heddle |

### Layer C (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T15 | **Codex as darn writer, replayed on a landed task through the real producer** | [`codex-writer-bench`](design-codex-writer-bench.md) | darn · docs | T19 | 1d<br>21h | codex, shuttle |

## Notes (2)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Codex as assistant Code-Review reviewer on real darn runs**<br>• Set Codex as the non-gating assistant reviewer on real runs and read what it adds beside the Claude reviewer | [`codex-assist-review-trial`](design-codex-assist-review-trial.md) | 11d<br>21h | codex, shuttle |
| R2 |  | **Codex support in levels: review, darn, then the rest of tapestry**<br>• Say lym supports Codex only at a level proven on real runs; the first work is a darn-writer bench | [`codex-support-levels`](design-codex-support-levels.md) | 7d<br>21h | codex, shuttle |
