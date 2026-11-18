# Support log — claude-account-pick-simple

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the question ledger.

Turn 1 — board and exploration.
Read the board entry, which carries the operator's simple design (2026-11-15) and the cancelled `claude-account-weights` record verbatim.
No `Decision for claude-account-pick-simple:` note exists; `lym loom peers` lists no peer and no parent commit; `adopt-build-resplit`, the dependency, is done.
Four explore subagents mapped the claude engine's launch and readers, hub geometry and adopt, config layers and usage-limit handling, and Claude Code's upstream facts.

Turn 2 — settled from the exploration.
- The claude engine reads `~/.claude` hard-coded for transcripts and registry; only the plugins lookup honours `CLAUDE_CONFIG_DIR`; `ScrubbedEnv` does not scrub it.
- The engine is not told the worktree today; the runner holds it.
- The `env` module allows `CLAUDE_CONFIG_DIR` and exports it ahead of the launch command; reed replays stored commands.
- Usage limits: the engine classifies the turn end with a reset; shuttle re-sends with a short backoff; the orch watcher waits for the reset.
- Upstream: the preserved-thinking page says foreign-organization thinking blocks are dropped silently, so a copied transcript resumes; no non-interactive usage read exists in 2.1.327.

Turn 3 — design picks (question ledger Q1 to Q10), then the three files written.

Turn 4 — lens review. Every lens returned a report.
- bound-breaker, F1 BLOCKING (resume check runs before `Prepare`, so a copy in `Prepare` comes too late): fixed; D6 moves the copy into the resume check.
- bound-breaker F2, interactions F1, existing-paths (stale copy on switching back): fixed; D6 brings the newest copy over the resolved one.
- walkthrough BLOCKING, bound-breaker F4 (restart through the stop verb ends the run as stopped): fixed; D7 relaunches inside the same run, and the stop-verb constraint is gone. Revised in review round 1: the relaunch goes through the stop verb's relaunch mode.
- existing-paths BLOCKING (live-session readers read the worktree's current account): fixed; D4 records the launched directory in the run directory and D8's live readers read it.
- walkthrough MEDIUM ×2, bound-breaker F3, interactions F3 (no launched account for a worktree with no entry on a listed directory; nowhere to keep it): fixed; D4 maps the effective directory to a listed account and records it.
- existing-paths MEDIUM (reader and namer engines built with empty `Dirs`): fixed; D8 has them built with the hub's `Dirs`.
- board-scope F1 (a pinned worktree waiting contradicts the board): fixed; a pinned worktree switches while its pin is out and returns after (D4, D7). Revised in review round 1: only a beams-pinned worktree switches.
- board-scope F2 (beams pin keeps beam-worktree workers from spreading): kept as an Open risk for the run's parent; pinning by role would break one account per worktree, a call the parent owns. Revised in review round 1: settled as an auto-pick (Q11).
- board-scope F3, interactions F4, bound-breaker F6 (prime under adopt and the limit switch): fixed; D3 and D9 say the prime is never picked and gets an entry only by `set` or a limit switch.
- board-scope F4, interactions F6 (one account with a pin; the scrub with no accounts): fixed in D1's closing sentences.
- interactions F2 (a removed account strands its transcripts): fixed; weight 0 retires an account and keeps its directory known (D1, D8).
- interactions F5 (a switch does not advance the counter): fixed; D7 increments.
- bound-breaker F5, existing-paths LOW (recreated worktree keeps a stale entry): fixed; the pick at making replaces any entry (D3).
- walkthrough LOW (picking for pinned worktrees skews the counter): fixed; pinned worktrees are not picked (D3).
- walkthrough LOW (double session-end usage record): fixed by the in-run relaunch, one run and one record (D7).
- bound-breaker F7 (merge dropping `blocked_until`): fixed in D5.
- bound-breaker F8 (agents can run `account set`): fixed; D7's bound says so.
- existing-paths LOW (opened sessions record no `blocked_until`): fixed; stated as a bound in D5.
- leanness 1 (auto-mode assumptions repeat decisions): fixed; each now only names its decision.
- leanness 2 (names are a plan hint): fixed; moved to Notes for the plan writer.
- leanness 3 (size in Constraints): fixed; moved to Goal.
- leanness 4 (seam twice in Constraints): fixed; merged.
- leanness 5 (Scope Out adopt bullet repeats D3): fixed; trimmed.
- leanness 6 (D2 git argument): fixed; cut to one clause, the rest is in Rejected alternatives.
- leanness 7 (D9 rationale repeats D1): fixed; reworded to the board's reason.
- leanness MEDIUM (orch resume through the engine unverified): fixed; walkthrough and existing-paths confirmed the orch resumes through the runner with a resume session id, and D6 now names that path.
- leanness LOW (relevant.md lists files known through subagents): rejected; those reads were this session's own exploration.

## Rejected alternatives

- The previous per-worktree record under the anchor with a transcript search on loss: the source of the cancelled design's size; a name-keyed cache plus D6's copy makes loss harmless.
- Keeping the account entry in a tracked file for adopt: account names are machine facts that never go into a pushed file, the target machine's accounts differ, and the entry decides no run's course.
- Picking at the first launch instead of at worktree making: would move worktrees made before the change; the board names worktree making as the moment.
- Moving the transcript instead of copying: a copy keeps the old directory's readers and a concurrent resume working; D8 takes the newest copy.
- Copying only when the resolved directory lacks the transcript: loses the turns taken elsewhere when a worktree switches back.
- A beams-pinned worktree waiting out a limit: contradicts the board's "waited out only when no account has headroom".
- A hub-pinned worktree switching on a limit: takes the hub onto an account the board's "this hub only on private account 1" excludes.
- A hash of the slug as the pick: cannot keep the long-run share once accounts are skipped.
- Pinning the orch and dobby by role rather than the prime and beam worktrees: breaks the board's one account per worktree (Q11).
- Restarting a limited session through shuttle's stop verb as it stands: ends the run as stopped under its waiter; D7 adds a relaunch mode that records no outcome.
- A per-account usage budget from lym's own transcript totals: the status line gives the real figures and limit hits give the reset.
- Storing the config directory in the entry: a name keeps one source of truth in the list.

## Review rounds

Round 1 (BLOCKING): every finding fixed; see `_lym/reviews/discussion/round-1-fixer-report.md`.
D7's relaunch is now a defined mechanism through the stop verb's relaunch mode and `Prepare`, ordered against `retryAPIError`;
only the beams pin switches on a limit; D6 gains a fresh-start fallback.

Round 2 (BLOCKING): every finding fixed; see `_lym/reviews/discussion/round-2-fixer-report.md`.
D7's relaunch keeps the strand's GUID, so every caller holding it reaches the relaunched run; its relaunching mark gets a bound, a sweep exemption and a stop path.
The relaunch resumes with a continue line and passes the startup gates; D6's copy record is carried into the run directory on the start path;
the orch restarts through its start path's relaunch branch; D5's forged-input bound covers D7's switch.

Round 3 (BLOCKING): every finding fixed; see `_lym/reviews/discussion/round-3-fixer-report.md`.
D7's relaunch writes the record before reed relaunches the pane, and the bound clears a mark over a live strand;
its startup wait runs with the events proof off, so the reused events file marks nothing started;
one running predicate reads a relaunching record as running for every reader, and the shed step stop stops it.

## Question ledger

- Q1 (auto-pick): where the per-worktree choice lives. Picked one hub-wide machine-local file under the Bolt's `.lym`, keyed by worktree name, holding account names (D2).
- Q2 (auto-pick): what of it goes into git. Picked nothing; it is a cache (D2).
- Q3 (auto-pick): pin keys. Picked `account_pins.hub` and `account_pins.beams`, the latter covering the prime and every beam; hub wins (D1).
- Q4 (auto-pick): when the pick runs. Picked worktree making on the machine, replacing any entry, never for a pinned worktree or the prime (D3).
- Q5 (auto-pick, factual, settled by docs): does a copied transcript resume under another account. Yes, with foreign thinking dropped silently (preserved-thinking page); manual check in D10.
- Q6 (auto-pick, revised after review round 1): does a pinned worktree switch on a limit. Picked yes for the beams pin, while the pin is out, returning after; a hub pin waits (D4, D7).
- Q7 (auto-pick, revised after review round 1): how a limited session continues after a switch. Picked a relaunch inside the same run as a resume under the same strand GUID, through the stop verb's relaunch mode and `Prepare`; the orch through its start path's relaunch branch (D7).
- Q8 (auto-pick): the headroom source. Kept the previous record's status line verb, simplified to the account name, plus a `blocked_until` from limit hits (D5).
- Q9 (auto-pick): the explicit switch verb. Picked `lym claude account set` and `show`, run in the worktree (D7).
- Q10 (auto-pick): removing an account without losing its history. Picked weight 0 as retired, its directory still known (D1, D8).
- Q11 (auto-pick): do card implementers in a beam worktree count as workers that must spread under the beams pin. Picked no: the beams pin covers every session of a beam worktree, reading the board's spreading workers as the task worktrees under the beams, by the board's "every session of one worktree runs under one account" (Auto-mode assumptions).
