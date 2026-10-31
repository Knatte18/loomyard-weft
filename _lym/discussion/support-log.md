# Support log — interactive-recipe

## Interview

The session ran autonomously: no operator turns.
Each turn below is a design question the writer settled from the board entry, the code and its own judgment.

### Turn 1 — What the board says

The entry `interactive-recipe` settles the recipe's name, the agent's role, its stencil's content, the verb-driven end, the run after the build segment and that the quiet notice and idle timeout are off while the agent waits.
No `Decision for interactive-recipe:` note exists. `lym loom peers` lists no peer and no parent commit.
Touching notes read: `interactive-idle-park` (builds on this, out of scope), `pr-continue-wakes-row` (PR-Review's wait, out of scope), `reed-status-buttons` (the trial case), `board-agent-button` (another interactive agent, opened by a button, not a run), `seed-starts-at-start` (sampler's first row is Preflight, so no conflict), `bolt-dir-rename` (depends on this task, no overlap).

### Turn 2 — Rows of the recipe

Settled on darn's shape minus Drift, with the build row replaced: Preflight, Record-Write, Sampler-Session, the Code-Review segment, Describe, PR-Review, Finalize, Friction-Reflect (D1).
PR-Review stays because only its resolver decides a pull request, and it passes on an empty landing.

### Turn 3 — How the perch is off by default

`profiles.yaml` is seed-only, so a `defaults.sampler` entry pointing at a profile with zero rounds would never reach an existing hub.
A `sampler.code_review` switch on the perch rows' `entry_gate`, default false, reaches every hub through the template (D3).
A profile without a sampler block refuses at seed, so the template's ladder profiles gain one (D2).

### Turn 4 — Who may record decisions

`decision add` refuses every task session but the parent orch and the arbiter.
The agent records what the operator settles, so the caller rule admits the Sampler-Session row's own strand with `--by operator` alone, while the row is current (D4).

### Turn 5 — How the agent runs and how the build ends

PR-Review already keeps an unawaited interactive session in a window of its own while a Go row waits on a flag the agent's verb writes; the sampler row follows that shape (D5, D6).
An unawaited session has no shuttle deadline, so the idle timeout is off by construction.
An entry stamp keeps a stale flag from ending a row re-entered by `goto`.
A dead strand halts with `lym loom resume`, since the agent is the whole work of the row.

### Turn 6 — Quiet notice and watchdog

Dobby's quiet notice is suppressed by a hard-coded list of waiting rows in `battencli`; the loom loop's step watchdog has no such list and counts a held turn end as no activity, so a sampler step would be killed after `step_idle_timeout_min`.
Both readers now read one list holding Beam-Await, PR-Review and Sampler-Session (D7).

### Turn 7 — Trials

Reed derives one tmux server per hub path and pins server-wide bindings from the newest build, so a dev build tried in this hub disturbs the operator's server.
Another hub has its own server; on lym's repository that is the sandbox hub.
A tmux-based trial outside lym uses `tmux -L` with a name of its own (D9).
Note `pr-continue-wakes-row` asks PR-Review's wait to wake at once on its verb; D6 gives the sampler wait that behaviour from the start, so the two waits differ until that note lands.
Note `interactive-idle-park` will build on this recipe's wait; D6's done record and entry stamp are where a park would hook in.

### Turn 8 — Lens review

All six forks returned a report. One fact was checked before judging: the Describe description gate checks only the file's format (`internal/summaryparser/description.go`), so a description of an empty diff passes it.

- interactions 1, existing-paths 1, walkthrough 1 (merged): Describe skipped under a zero Code-Review budget leaves no description — fixed, Describe carries no `entry_gate` (D1).
- interactions 2: Finalize's reroute and `CodeReviewOff` read the budget, not `sampler.code_review` — fixed, every perch-off reading follows the rows' entry gate too (D3).
- interactions 3, walkthrough 2, bound-breaker 5 (merged): a pause could halt the row on a session it stopped — fixed, covered wakes judge nothing and the uncovered wake resumes first (D6).
- interactions 4: the admitted strand after a resume — fixed, the row re-records the strand and D4 reads the current one.
- interactions 5: dirty-tree way forward wrong for a failed weft commit — fixed, the check covers the code checkout only, which a `_lym` commit never dirties (D6).
- interactions 6: copied profile perch values inert — fixed, `light`, `solid` and `wide` set `code_review` true, `speed` leaves it false (D2).
- existing-paths 2, 3 (merged): tree-scope, budget and record-check seams are wired only for a `DarnWrite` recipe — fixed, wired by the rows that need them (D6, Scope).
- existing-paths 4: refusal after the row ended can never reach the stopped agent — rejected, harmless and within the bound.
- existing-paths 5: the list's home must be importable by both readers — fixed, stated in Notes for the plan writer.
- walkthrough 3, bound-breaker 4 (merged): the entry stamp's source unstated — fixed, the stamp is the count of the row's Done history entries (D6).
- walkthrough 4: the verb is accepted while the row is halted — fixed, stated as a bound (D6).
- bound-breaker 1: any reed op of a dev build re-pins bindings, not only `lym reed` — fixed, the rule forbids running a dev build in this hub (D9).
- bound-breaker 2: Describe's gate on an empty diff unverified — fixed, researched and recorded under Constraints; acceptance criterion extended.
- bound-breaker 3: docs-only diff runs no tests only when the hub lists trees — fixed, D1 reworded.
- bound-breaker 6: a closed window may be reopened by the stop chain or `lym reed resume` — fixed, accepted and stated as a bound (D6).
- bound-breaker 7: a pause holds a step and never ends it — fixed, D7 bound reworded.
- board-scope 1 (class decision): does another hub plus a stencil rule meet "a tmux server of its own" — recorded under Open risks as a question for the run's parent.
- board-scope 2: D7 reaches PR-Review and Beam-Await beyond "this recipe" — rejected, no entry owns that behaviour and the two readings must agree; disclosed under Open risks.
- board-scope 3: the sampler wait wakes at once while PR-Review's does not — rejected as no conflict; noted in Turn 7.
- leanness 1: D3 rationale repeats the support log — fixed, trimmed.
- leanness 2: D7's last sentence states the fact three times — fixed, kept once under Constraints.
- leanness 3: Constraints restate PATTERN entries cited inline — fixed, the inline-cited ones dropped from Constraints.
- leanness 4: D8 rationale is deliberation — fixed, shortened.
- leanness 5: keep D9's evidence — no change needed.
- leanness 6: garbled "PATTERN-free" phrase in Scope — fixed.
- leanness 7: verb-name assumption duplicates D6 — fixed, moved to the question ledger.
- leanness 8: the two note observations are no risks — fixed, moved to Turn 7.
- leanness 9: plan-writer notes overlap `relevant.md` — fixed in part, the darn-wiring bullet dropped; the rest kept.

## Rejected alternatives

- A `defaults.sampler` profile with zero Code-Review rounds as the off switch: `profiles.yaml` is seed-only and never reaches an existing hub.
- An awaited shuttle run with a very long deadline for the agent: still a deadline, and the seat engine's turn-end and hold machinery is built for a run that finishes by writing output files.
- Reusing Discussion-Write's interactive mode: its stencil binds the agent to exploring and writing a record, never to the work.
- Disarming the step watchdog for the whole sampler run: Describe, the perch and Finalize still benefit from it; the list exempts only the waiting row.
- Keeping the wait after a dead strand and starting no session, as PR-Review does: a sampler row with no agent does nothing, so it halts and says how to resume.
- A run-owned reed server through `TMUX_TMPDIR` or a server-name override: reed's per-worktree state and the hub's watchdog daemon would still meet the trial's reed in the same worktree; a separate hub isolates both.
- Building a throwaway hub with `lym fabric clone`: it binds real remotes, so a trial could push to them.
- Letting the sampler agent create board tasks because the operator is present: the board's run-agent rule and the Discussion rules forbid any decision that lets a run's agent create or promote a task.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): Which rows does sampler carry? Darn's without Drift, with Sampler-Session as the build row. Rationale: the operator's sequence, and Drift guards code regressions a prose run rarely has.
- Q2 (resolved, auto-pick): How is the perch off by default? A `sampler.code_review` key, default false, on the perch rows' `entry_gate`. Rationale: reaches every hub through the template.
- Q3 (resolved, auto-pick): Does a task naming a profile need a sampler block? Yes; the template's ladder profiles gain one equal to their darn block, with no sampler default.
- Q4 (resolved, auto-pick): Who may run `decision add` on a sampler run? Also the Sampler-Session row's own strand, with `--by operator` only, while that row is current.
- Q5 (resolved, auto-pick): How is the session kept? As PR-Review's: unawaited, reused, resumed, no deadline.
- Q6 (resolved, auto-pick): What ends the build segment? `lym loom sampler-done` writes a stamped done flag; the row ends on a clean tree and an intact record.
- Q7 (resolved, auto-pick): What happens when the agent's strand dies? Stuck with Halt naming `lym loom resume`.
- Q8 (resolved, auto-pick): How are the quiet notice and the watchdog kept off? One list of operator-waiting rows read by both, including Beam-Await and PR-Review.
- Q9 (resolved, auto-pick): How does a trial get a tmux server of its own? Another hub, the sandbox hub on lym's own repository, and `tmux -L` for tmux outside lym; no new mechanism.
- Q10 (resolved, auto-pick): Does batten open the window regardless of `batten.open_window`? No, batten stays the same for every recipe.
- Q11 (resolved, auto-pick): Is "started only when the operator means to work with it" a new start rule? No, it is the reason for D7; releasing the task is the go-ahead.
- Q12 (resolved, auto-pick): Default model of the agent: `opus[high]`.
- Q13 (resolved, auto-pick): Does the build row run a verify gate? No; Finalize's gate checks the merged tree.
- Q14 (resolved, auto-pick): Name of the verb: `sampler-done`, beside `pr-continue`.
- Q15 (resolved, auto-pick, after lens review): Does Describe keep darn's budget gate? No; the sampler agent writes no description, so Describe always runs.
- Q16 (resolved, auto-pick, after lens review): What is the entry stamp? The count of the row's Done entries in the status history.
- Q17 (open, for the run's parent): Does another hub plus a stencil rule meet "a tmux server of its own" for a trial? Recorded under Open risks.
