# runs-overview — support log

## Interview

The session ran autonomously: no operator answered, and every pick below is an auto-pick recorded in the Question ledger.

1. Read the board entry and the related entries (`orch-split`, `reed-multi-window`, `extra-agent-strand`, `operator-shortcuts`, `ide-prime-setup`, `lyx-halt`).
   `batten-notice-trim` and `batten-windows-hidden` are no longer on the board; their commits (17b09e97f, a35c8c2b1) are in history.
2. Explored reed's layout rule, Selvage and bindings in `internal/reedengine/doc.go`, `render`'s anchor vocabulary, batten's notice code, the batten `--window` path and `termwindow`, the loom status strand bootstrap, shed's status file and history entries, and `reed attach`'s exit-code propagation.
3. Scope: built every merged section of the entry (overview, sizing, Selvage keys, checklist, run windows), with the notice log because the entry states it "either way".
4. Overview placement and ownership: a batten pane verb in a `runs` strand that `lyx orch start` ensures; a top anchor makes its position independent of insertion order.
5. Sizing: three kinds (positional default, content, sharing) plus a top anchor; content rows pushed by the pane's own process through one engine operation.
   Shared strands stack in rows, though `reed-multi-window`'s 2026-10-05 note prefers one tmux window per strand on a small screen; the board entry's 2026-10-08 sizing proposal leaves rows open, and the depth idea is that note's own work (A5).
6. Checklist: replaces the `loom-status` strand's content and is kept on both drivers, since the llm child driver is the default.
7. Run window: close on attach exit 0, a bounded wait for the pair directory on a non-zero exit, else keep the shell.
8. Lens review by forks: all six lenses returned; findings and dispositions:
   - board-fit: rows contradict `reed-multi-window`'s depth note — fixed, named in the Interview (item 5), A5 and Open risks.
   - board-fit: `orch-split` unnamed — fixed, Scope Out names it.
   - board-fit: `extra-agent-strand` as a future consumer — fixed, Notes.
   - board-fit: `own-window` stays reserved — fixed, D3.
   - board-fit: `lyx cleanup` vs the notice log — fixed, Open risks; moved to Constraints in round 1.
   - board-fit: "in flight" shared with `alignment-agent` — rejected: that note is unbuilt and defines nothing to share yet.
   - board-fit: Related entries cite landed tasks — no change, consistent.
   - leanness: assumptions repeat decisions — fixed, A-list cut to picks with a pointer.
   - leanness: Goal repeats Scope — fixed.
   - leanness: D2 attach rationale is deliberation — fixed by turning it into the stated departure; moved to Rejected alternatives in round 1.
   - leanness: test-budget and pane-binary constraints redundant — fixed, removed.
   - leanness: redraw risk not actionable — fixed, removed.
   - leanness: D5 declarations stated thrice — fixed, D1 and D7 point at D5.
   - leanness and scenarios: silent 30-second window wait — fixed, D10 prints first.
   - goal-scope: attach command handed over by `batten-notice-trim` — fixed, the departure is recorded under Rejected alternatives.
   - goal-scope: checklist size and llm-run presence unlisted — fixed, quarter share in D5, A3.
   - goal-scope: notice log path unsurfaced — fixed, `lyx batten` help names it (D8).
   - goal-scope, premises, scenarios: stale non-`done` runs stay forever — fixed, D2 needs a pair or Worktree-Create.
   - goal-scope: top anchor not linked to the board — fixed in Scope.
   - pattern-fit: `battenshed` cannot import `shedrun` — fixed, path told through deps (D8).
   - pattern-fit: reuse `shedverbs.RenderStatusLine` — fixed, D7.
   - pattern-fit: spawn color unstated — fixed, no segment (D1, D7).
   - pattern-fit: sandbox coverage — fixed, Constraints.
   - pattern-fit and premises: lock-free read safety — fixed, D2 cites the atomic write.
   - pattern-fit: Selvage marker display-only — fixed, D9.
   - premises: top pane's title row eats a line — fixed, D3.
   - premises: polling cost over many `done` runs — fixed, mtime check in D2.
   - premises: Alt+z condition and batten windows — fixed, D9 and an acceptance criterion.
   - premises: `detach-on-destroy off` — fixed, Open risks.
   - scenarios: checklist overflow hides the running row — fixed, D4.
   - scenarios: missing child status reads as a fault — fixed, D2.
   - scenarios: no starting count, persistence — fixed, D3.
   - scenarios: first push refused before recording — fixed, retry in D4.
   - scenarios: bad counts and wrong caller — fixed, D4 bound with the pane id.
   - scenarios: stale-binary overview — fixed, D1 replaces another build's strand; a crashed overview waits for the next orch start, accepted.
   - scenarios: sharing with positional strands — fixed, D3 mixed rule.
   - scenarios: D10 bound overstated — fixed, names `reed down` and reaps.
   - scenarios: concurrent and retried appends — fixed, D8.
   - scenarios: transient read errors — fixed, two-poll rule in D2.
   - scenarios: renamed rows in history — fixed, D7.
   - scenarios: zoomed Selvage across re-apply — fixed, D9 and an acceptance criterion.
   - scenarios: sort key with no seed — fixed, D2.

## Rejected alternatives

- **Overview in its own window** (2026-10-02 and 2026-10-05 choice): replaced by the operator's 2026-10-12 layout.
- **Notice log lines inside the overview pane**: the pane grows with every notice and stops being small; the log file serves the same purpose.
- **Full attach command on each overview line**: too wide for one line per run; the session name reaches the run from the attached client, and the notice log carries the command.
  This departs from the board entry's note that the overview carries each run's attach command (handed over from `batten-notice-trim`): an attached operator reaches a run by its session name through Alt+Left/Alt+Right or the session buttons, and an operator outside the reed session gets the full command from the notice log (D8).
- **Columns for sharing strands, or a rule by window width**: the board calls columns a broader revision; rows keep the render layer's stacking and work on a small laptop screen.
- **Reed choosing a strand's kind by role**: reed would have to interpret strand names, which it never does today.
- **Reed polling the pane content to learn its line count**: costs a tmux round trip per poll per strand and guesses at what the process meant.
- **A `lyx reed` verb for sizing**: no operator or other module needs it.
- **A second status strand for the checklist next to `loom-status`**: two status panes in one pair for the same file.
- **Checklist only on go-driven runs**: the default child driver is llm, so it would almost never show.
- **Window closes only when the pair directory is gone**: the session ends before the worktree is removed, so the check races the teardown; exit 0 is the earlier, reliable signal.
- **Detach keeps the window**: a dedicated run window has no other use, and detaching is how the operator ends watching.
- **Reed.yaml keys for the overview's share and minimum**: no one asked for a knob.

## Review rounds

- Round 1: eight findings, all fixed: D7's row time built on end times, the last-strand rule and the anchor-kind table in D3, D8's append trigger, D2's and D7's unstated states, the cleanup risk as a Constraint, D2's departure and A5's narrative moved here, and acceptance criteria for D1, D2 and D7's untested behaviors.
- Round 2: three findings, all fixed: D7's row time counts halt waits as wall time between stamps and drops the wait before a `goto`, D7 gains a halted box for paused, blocked and awaiting runs, and D8's append is once per batten process, logged again after a restart.

## Question ledger

- Q1 (auto-pick): Notice log in the pane or only run lines? Only run lines; the log is a file (D8).
- Q2 (auto-pick): Columns or rows for parallel agents? Rows (D3).
- Q3 (auto-pick): Who sets a strand's kind? The spawning module on its add or launch spec (D3, D6).
- Q4 (auto-pick): How does the overview tell reed its line count? An engine operation called by the pane's process on change, which re-applies the layout (D4).
- Q5 (auto-pick): What does "age" mean on a line? Both stage time and run age (D2, A1).
- Q6 (auto-pick): Which runs are "in flight"? Every prime batten run not `done` whose pair exists or that is in Worktree-Create (D2).
- Q7 (auto-pick): Where does the checklist live? The pair's existing `loom-status` strand, on both drivers (D7).
- Q8 (auto-pick): Does an operator detach close the run window? Yes (D10, A6).
- Q9 (auto-pick): Does `lyx orch stop` remove the overview? No (D1).
- Q10 (auto-pick): Does Selvage sit first or last in the Alt cycle? Last, its physical position (D9).
- Q11 (open, for the plan): tmux attach exit statuses on session kill and server shutdown (Open risks).
