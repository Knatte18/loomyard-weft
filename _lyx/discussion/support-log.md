# Support log — shuttle-state

## Interview

Autonomous run: no operator answered; every question below was settled by the agent from the board entry, the related entries and the code, and is ledgered as an auto-pick.

1. Read the board entry `shuttle-state` (four merged sections plus issue #429) and the related entries `creel`, `lifecycle-waits` (done), `runs-overview`, `two-agent-producer`, `agents-no-scripted-edits`, `verify-watchdog`.
2. Checked what landed since the entry was written: `shuttle-turns` (35e36281e) made the Stop payload authoritative, holds every turn end without output and removed the asking outcome and the `shedadapters` "webster master is asking a question" branch; `lifecycle-waits` (fd5b0f58e) moved batten's quiet judgment onto `ReadAgentActivity` with the transcript API-error marker.
3. Checked Claude Code 2.1.294's binary for hook names: `StopFailure` exists, with `error`, `error_details` and `last_assistant_message`; `Notification` carries `idle_prompt` and `permission_prompt` types; `SessionEnd` exists. This answers the board's "whether Claude Code has a hook for an API error is to be checked".
4. Scope question: does this task include the bounded resume and the first consumer? The board's "Built in small steps" makes this task step 2 (shadow mode) and each consumer its own task; resume acts on the state, so it is out.
5. Where is the state computed? A pure reducer over the events file the hooks already write, so any process reads the same answer, with logging at the three sites that guess today.
6. How to tell `asking` from `idle-stalled` without reading the message text: by explicit ask signals, and by run mode for a missing-output turn end.
7. #429 after `shuttle-turns`: the halt is gone, but a waited-out shell still turns into a held turn end and a parent notice after 10 minutes; decided payload-reported shells wait to the run Timeout.
8. Stuck-as-question: traced to the removed `shedadapters` branch; the card-batching run predates `shuttle-turns`, so the task pins the behavior.
9. Permission setup: found `PATTERN-no-denied-recovery` naming `agent-permission-setup` as the owner of the denied list; this task's deny table takes that role.
10. Python deny in a Python target repo: a default-on config switch.

## Rejected alternatives

- **A new `EventKind` per hook in `ParseEvents`.** The wait loop classifies a batch by its last event, so a turn-start line arriving after a Stop would change verdicts; shadow mode must change none.
- **Stamping events with the observer's poll time only.** The orch session has no wait loop, and a crashed wait process would stamp late; a hook-side `lyx_at` is the source.
- **A Go hook (`lyx shuttle record-event`) instead of shell.** Ties every hook to resolving the `lyx` binary inside Claude Code's hook shell and adds a process per hook; the POSIX one-liner already works under git-bash.
- **A separate hooks file beside `events.jsonl`.** The board asks for the same event file; one ordered file is what lets the reducer fold turn starts and ends in order.
- **The Wait loop as the sole writer of a persisted `state.yaml`/history file.** The orch session has no Wait; deriving from the timestamped events file gives every reader the same history with no new writer.
- **Classifying a no-output turn end as `asking` in every mode.** It conflates a stalled autonomous agent with a question; without text-guessing the run mode is the only real signal.
- **Pane liveness in the reducer.** The board makes the pane a cross-check only; the registry pid is the process.
- **Building the bounded resume now, gated off.** It is a consumer, and the board defers consumers to their own tasks.
- **Keeping `background_shell_wait_min` for payload-reported shells and stopping the shell on expiry (#429's third option).** Kills or orphans the very measurement being waited on; with an authoritative payload a running shell is live work.
- **Waiting on leftover shells when every output file exists.** Delays the gate by up to 10 minutes for work the step no longer needs (`shuttle-turns` PR-Gate finding).
- **Rebuilding the stuck-classification path.** The path that produced it no longer exists.
- **Denying only inline scripts (`python -`, `python -c`, heredocs).** The operator's rule is "lyx agents never use Python".
- **An unconditional python deny.** Breaks agents in a Python target repo that must run `python -m pytest`.
- **Matching `python` anywhere in the command.** Denies `grep python file` and paths; command position is the bound.
- **Reading or editing the operator's `~/.claude/settings.json`.** Not lyx's file; agents run in bypass, and lyx's texts already avoid denied commands.
- **A new PATTERN entry for "only shuttle's model reads session state".** No consumer reads it yet and today's guesses remain; the invariant belongs with the first consumer.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick, resolved): Bounded resume of `idle-stalled` in scope? No; the board's step plan defers every consumer, and resume acts on the state.
- Q2 (auto-pick, resolved): Hook line format? Same events file, one write per line, hook-side `lyx_at` inserted as the first field; lines without it read at first-seen time.
- Q3 (auto-pick, resolved): New signals through `ParseEvents`? No; a separate optional parse capability, so the wait loop sees no change.
- Q4 (auto-pick, resolved): `asking` vs `idle-stalled` for a no-output turn end? Run mode decides: interactive `asking`, autonomous `idle-stalled` `no-output`; explicit ask signals are `asking` in both.
- Q5 (auto-pick, resolved): Liveness source? Claude's session registry pid with `procStart`; no entry is `unknown`.
- Q6 (auto-pick, resolved): Shadow sites? Wait loop, orch watcher's idle probe, batten's activity reading; log only.
- Q7 (auto-pick, resolved): A CLI to read the state? `lyx shuttle state`, read-only, current worktree's live runs.
- Q8 (auto-pick, resolved): Does a held mark suppress batten's quiet notice? No; `waitLive` counts gate and shell waits only.
- Q9 (auto-pick, resolved): #429 wait rule? Payload-reported outstanding work waits to the run Timeout with no held notice; `background_shell_wait_min` stays for transcript-reported shells only.
- Q10 (auto-pick, resolved): Outputs present with leftover shells? Done at once, shells named in `ExpiredShells`; forks and awaited prefixes still wait.
- Q11 (auto-pick, resolved): Stuck-as-question fix? Already removed by `shuttle-turns`; pin with a test.
- Q12 (auto-pick, resolved): Python deny scope? Command-position `python`/`python3`/`python3.N`, every lyx-spawned session, default on, `claude_deny_python: false` per repo.
- Q13 (auto-pick, resolved): Owner of `PATTERN-no-denied-recovery`'s list? This task's deny table; the background file is updated to say so and the list gains `python`, `python3`.
- Q14 (open, risk): Real `StopFailure` and `Notification` payloads are not yet captured; the `llm`-tier scenario captures `Notification`, and `StopFailure` stays schema-derived until a real API error leaves one.
- Q15 (open, risk): Whether `UserPromptSubmit` fires for a self-started turn (background completion wake); recorded under Open risks.
