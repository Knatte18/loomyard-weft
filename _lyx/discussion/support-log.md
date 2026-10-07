# Support log — shuttle-turns

## Interview

Autonomous mode: no operator answered; every question below was settled by reading the code and run artifacts, then picked.

1. Why does Wait not take the #411 Stop?
   The board's run dirs were gone with their worktrees.
   The orch's own diagnosis fork (in the orch transcript `330d7c24`) found no cause.
   Reading `background.go` showed that a Stop becomes `EventWaiting` when the transcript holds a background launch with no later `<task-notification>` user message, and a gated run never takes a waiting turn end until a shell expires.
   The burler transcripts of both runs hold a backgrounded `go test` whose notification was `absorbed_mid_turn`: it appears only as `queue-operation` lines and a `queued_command` attachment.
   Root cause settled (D1).
2. Which signal tells the truth about outstanding work?
   The hub's events files show `background_tasks` entries of type `shell` and `subagent`, so the payload is complete; it becomes authoritative, with the transcript as a fallback only (D2).
3. What bounded exit replaces SIGQUIT?
   With the phantom gone, the existing shell expiry and run deadline bound every wait; a trace line names what a wait waits on (D3).
4. How is a held turn end surfaced?
   Through a notifier seam wired to the orch notice queue, once per held turn end, silent for interactive runs (D5).
5. What happens to `asking` everywhere it is consumed?
   It retires with `AwaitOperator`, and so do the caller branches and treadle's asking triage (D4, D7).
6. Does `lyx shuttle run --interactive` need `asking` to stay terminal?
   `spec.go` says so, but no production caller uses the verb; the hold applies there too (D7).
7. Where does the message tail live?
   In the stencils, in `lyx shuttle send`, and in the hold notice; gate re-prompts excluded (D8).
8. Background-shell half of #407?
   The hold covers it: an expired shell's turn end without outputs is a hold, not a stop (D9).
9. Is the crucible rewrite in scope?
   No: live scenarios are `llm`-tier tests; `crucible/` and Hardener stay with the parked `hardener` note (D10).

## Rejected alternatives

- Keep the transcript scan as an equal signal and only teach it `queue-operation` lines: fixes today's shape but keeps reading undocumented lines first, the class of misread this task exists to end.
- Make forks expire like shells to bound the wait: would count a turn end while a real fork still works.
- A per-spec knob to keep `asking` terminal for `lyx shuttle run` or treadle: no production caller needs it (YAGNI), and it would keep the hazard alive.
- Notify through `SendMessage` from Go: no Go route exists into a Claude session but the orch watcher's queue (`PATTERN-orch-pane-single-writer`).
- Kill an expired background shell: lyx does not own the agent's shells, and killing one could destroy work the agent still needs.
- Add the Notification hook as a signal: no observed misread needed it, and lyx agents run with permissions bypassed, so its main use, a permission prompt, does not arise.
- Rewrite `crucible/` for loomyard now: a review-and-fix loop for unknown defects, owned by the parked `hardener` note; the scenarios here have known answers and fit the `llm` test tier.
- Tail on every `Run.Send`: gate re-prompts are instructions at a turn boundary, where "answer briefly" is wrong.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Root cause of #411/#415 — resolved from transcripts (D1).
- Q2 Authoritative outstanding signal — auto-pick: payload `background_tasks` when present (D2); rationale: complete and official.
- Q3 Bounded exit — auto-pick: existing expiry and deadline plus a trace line (D3).
- Q4 Notice route and frequency — auto-pick: notifier seam to the orch queue, once per held turn end, autonomous runs only (D5).
- Q5 Fate of `OutcomeAsking`/`AwaitOperator` and caller branches — auto-pick: retire all, including treadle's asking triage (D4, D7).
- Q6 `lyx shuttle run --interactive` — auto-pick: holds like every run (D7); rationale: no production caller.
- Q7 Legacy `asking` run.json at deploy — auto-pick: attach when the strand is live (D6).
- Q8 Tail placement — auto-pick: stencils, `lyx shuttle send`, the notice; not gate re-prompts (D8).
- Q9 Expired background shell — auto-pick: hold instead of stop; no shell killing (D9).
- Q10 Crucible scope — auto-pick: out; live scenarios as `llm` tests (D10).
- Q11 Corpus fixtures for runs whose events were deleted — auto-pick: reconstruct `events.jsonl` from the transcript and mark it (D10).
- Environment: the board's run dirs (`976daeeb…`, `d9a47c20…`) no longer exist; the transcripts under `~/.claude/projects/` were used instead. Nothing was repaired.
