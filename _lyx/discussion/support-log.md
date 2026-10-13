# Support log — tender

## Interview

Autonomous mode: no operator answered; every pick below is the writer's, and each is listed in the question ledger.

1. Read the board entry and the related entries `orch-split` (concurrent-run cap, Go run driver), `creel` (pause holds deliveries; future owner of pane delivery), `spawn-audit-log` and `weaver`.
   `reed-window-order` is not on the board; its `internal/selfexec` and the status panes' use of it are in the tree.
2. Explored the pause path: `pausecli` mounts `pause` and `resume` as two root commands and installs the network gate; `pauseengine` holds the sequences; `pauserecord` holds the record, the waits and the gate.
3. Traced #606: `pauserecord.Network` holds inside `githubclient`'s `authRT.RoundTrip`, which runs under the http client's 30 s timeout and the caller's own 30 s context (`landingshed`'s `publishGitHubTimeout`).
   `gitexec` already holds before starting its deadline.
   The alignment session's own deadline is already pause-aware (`AwakeSince`).
   A GitHub deadline error is already transient; the loom drive rules re-step a transient failure, then go to the tackler.
4. Listed live processes: `orch watch`, `reed attach`, `reed watchdog` and loom step loops ran a deleted binary; status panes, the watchdog's retire, batten's binary re-step and the drive rule's hand-over already handle a new build.
5. Settled scope: pause, resume, revive and status in; drain, the run cap and a restart verb out; orch watch self-restarts; reed attach execs tmux.
6. Spawned the six lens forks in one message before writing, per the procedure.
   The `orch watch` and `reed attach` processes seen on a deleted binary had run for days; the stale watchdog started at 12:22 on 2026-10-13, before its retire landed (a90ef61d1, 13:26), so it does not show the retire broken.
7. Lens review, every fork returned:
   - leanness: Auto-mode assumptions repeated decisions — fixed, now a pointer to D1–D9 and the ledger.
   - leanness: second open risk repeated D6 — fixed, rewritten around the merged deadline.
   - leanness: Scope Out justification for mid-work processes — fixed, shortened.
   - leanness: Constraints restated D5's bound — fixed, points at D5.
   - leanness: Notes repeated D5's rationale — fixed, file pointer only.
   - leanness: D8 narrative "seen for days" — fixed, moved to item 6.
   - leanness: only one fork was spawned — rejected: all six were spawned in one message and all returned.
   - board-fit: `creel` not named — fixed, Constraints names it and keeps revive on pause's seams.
   - board-fit: stale watchdog unexplained — fixed in item 6: it predates the retire.
   - board-fit: `spawn-audit-log` overlap — fixed, named in Notes.
   - pattern-fit, scenarios: which binary path the listing compares — fixed in D7: base name of a deleted exe, Go bin and `.dev-bin` alike.
   - pattern-fit: extension placement against the leaf rule — fixed in D6: logic in `githubclient`.
   - pattern-fit: `PATTERN-agent-name` missing — fixed, added to Constraints.
   - pattern-fit: `PATTERN-refusal-way-forward` misapplied — fixed, removed; `PATTERN-no-denied-recovery` kept.
   - pattern-fit: re-exec not testable end to end — fixed, criterion tests through the exec seam.
   - pattern-fit, premises, scenarios: exec skips post-run — fixed in D9: flush and log first, run directory left to the sweep.
   - goal-scope, scenarios: second #606 requirement deferred to open risks — fixed in D6 with a re-step criterion.
   - goal-scope: restart dropped without saying so — fixed, Scope Out states the narrowing and the way forward.
   - goal-scope, premises, scenarios: `pause --status` references — fixed in D3.
   - goal-scope: exec'd attach ends a shortcut sequence — fixed in D9 with a guard bound.
   - goal-scope: extension covers one request only — fixed, bound stated in D6.
   - scenarios, premises: watch lock dropped across the exec (premises BLOCKING) — fixed in D8: the gap and its outcome are bounded, with a race criterion.
   - scenarios: orch revive notices not deduped — fixed in D5.
   - scenarios: revive racing a pause — fixed in D5 with a criterion.
   - scenarios: outer wall-clock budgets — fixed in D6; `publishretry` holds no time budget.
   - scenarios: pause after admission — fixed, criterion added.
   - scenarios: `pause --status` should refuse with a way forward — rejected: no alias by decision, and cobra's unknown-flag error with `lyx tender --help` suffices for an operator verb.
   - scenarios: agent reviving its own pair — fixed, noted in D5.
   - premises: client timeout and caller deadline merge in one context — fixed in D6: the transport applies the client timeout itself.
   - premises: stale processes sit in the prime, cwd cannot attribute a pair — fixed in D7, no pair attribution; the cwd open risk is dropped.

## Rejected alternatives

- Keep `pause` and `resume` top-level and add `tender` beside them: leaves two pseudo-modules in `lyx --help`, the problem the board names.
- Rename `pauserecord` too: a leaf imported by many packages, and its name still fits.
- Drain: pause with grace already makes the hub safe to suspend, and a deploy is already absorbed by binary re-step, drive hand-over and self-restart; drain would add a new hold point to every step loop for no case the existing tools leave open.
- A concurrent-run cap in tender: tender starts no runs; the cap belongs with whatever starts them, `orch-split`'s run driver.
- A `tender restart` verb that signals or kills stale processes: the processes that can restart safely do so themselves, and the rest must not be restarted mid-work.
- The deploy (`update-plugins.sh`, batten's Land deploy) restarting every process: couples deployment to process management and cannot know what is mid-work.
- For #606, starting the deadline only at each call site after a hold: every GitHub caller would need to know about pauses; moving the deadlines in the transport covers all of them.
- For #606, reclassifying a held deadline error: it is already transient; the defect is that the pause consumed the budget.
- Classifying stale processes (self-restarting, mid-work, other): a table of command paths to keep in step with the CLI, for a listing the operator reads directly.
- Restarting `reed attach` via selfexec: the process does nothing but wait on tmux; exec'ing tmux removes it.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Which ideas belong in tender? Auto-pick: pause, resume, revive, status (with the stale listing). Drain and the run cap out. Resolved.
- Q2 Package names? Auto-pick: `tendercli`/`tenderengine` renamed from `pausecli`/`pauseengine`; `pauserecord` kept. Resolved.
- Q3 Shortcuts? Auto-pick: `pause` and `resume` only. Resolved.
- Q4 Where does `pause --status` go? Auto-pick: into `tender status`, no alias. Resolved.
- Q5 #606 mechanism? Auto-pick: extend the transport's deadlines by the held time; no reclassification. Resolved.
- Q6 Who may revive? Auto-pick: the same rule as `pause`. Resolved.
- Q7 How does revive reach the orch? Auto-pick: an `orchengine` notice, per `PATTERN-orch-pane-single-writer`. Resolved.
- Q8 Which processes restart on a new binary? Auto-pick: `orch watch` re-execs; `reed attach` execs tmux; mid-work processes are listed only. Resolved.
- Q9 Restart verb? Auto-pick: none. Resolved.
- Q10 How is a hub process found for the stale listing? Auto-pick: cwd under the hub on Linux, matching a deleted exe by base name; no pair attribution. Resolved.
- Q11 How does the orch watcher keep one instance across its re-exec? Auto-pick: re-acquire the lock and accept the bounded gap, rather than handing the lock descriptor across the exec. Resolved.
- Q12 Does `pause --status` get a refusing alias? Auto-pick: no; references are updated. Resolved.
