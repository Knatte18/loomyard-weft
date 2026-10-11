# Support log — fork-wait-noise

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the ledger.

1. Read the board entry and the related entries `orch-token-spend`, `parent-review-cold-cache` and `twb-combo-1` (a bench entry whose item 1 covers a background gate shell cut off at a turn end; it is graded against an older base and is not live work to align with).
2. Read the implementer and verify-fix bodies, the Master template, the orch role stencil, `lyx shuttle await`, `lyx webster await-batch`, the gate verb's doc and the claudeengine pane-line and steer code.
   Found: the fork backgrounds every gate step because the body says so; the raw-go steer every lyx session gets says the same; the gate steps `renderCardGates` emits are already relative; "run from `{{.worktree_root}}`" invites `cd`.
3. Challenged the problem: the board suggests a capped in-turn wait on the gate's result.
   That needs a result record the gate does not write today, and a way for the waiter to tell the run it just started from a stale one.
   The cause of the turn end is the backgrounding itself, so removing it is the smaller fix: a foreground gate never ends the turn.
   Claude Code's 10-minute ceiling on a foreground call is what the backgrounding guards against; claudeengine already sets Claude Code variables on the pane line, so raising `BASH_MAX_TIMEOUT_MS` there removes the guard's reason.
4. Settled where the rule lives: the steer reaches every lyx session and its forks, so it holds the number; stencils point at it.
   The orch's fresh non-fork agents do not see it, so the orch's brief points at it.
5. Found that Master's step 4 sends any fork notice to `record-batch`, which would misclassify an interim notice; added a guard line for the fallback case.

## Rejected alternatives

- Background gate plus `lyx shuttle await` on a gate result marker (the board's example): needs a new result record in `lyx gate test` and a start-versus-stale race between the background call and the wait; it keeps the fork's cache warm on gates over five minutes, but the gates in the reported run each took a few minutes, and the foreground gate needs no new state.
- A detached `lyx gate start` with `lyx gate await <id>`: removes Bash backgrounding but leaves a gate process outliving its strand and holding a slot, against the tied-child lifetime in `internal/proc`.
- A PreToolUse deny on `run_in_background` inside forks: forks share Master's settings, and Master needs backgrounding for `recover-batch`; telling a fork from its parent in the hook adds machinery the stencil rule makes unnecessary.
- Leaving Master's step 4 as is: an interim notice routed to `record-batch` would fail a batch whose fork is still working.
- Restating the timeout number in every stencil: drifts from the constant; a pointer at the steer keeps one source.
- A PATTERN entry for the foreground rule: it is agent guidance pinned by claims tests, not a code-structure invariant.
- A configurable ceiling in `shuttle.yaml`: no one asked for a per-hub value; one constant suffices.
- A ceiling below `step_idle_timeout_min` instead of counting forks as activity (D9): the quiet stretch a waiting Merriam shows is the whole batch, not one call, so no ceiling keeps a long batch alive.
- Counting an outstanding fork as live activity in `shuttleengine`: it would read a hung fork as live until the run deadline; reading the fork's own transcript keeps the idle kill working on a fork that stopped writing.

## Review rounds

_No rounds yet._

## Question ledger

- Q1: Foreground gates with a raised ceiling, or a background gate with an in-turn await? — Resolved, auto-pick: foreground (D1). Smaller change, no new state, no race.
- Q2: What ceiling? — Resolved, auto-pick: one hour (`3600000`); covers the 300 s slot wait plus a tagged run, and anything longer is a hang.
- Q3: Where does the gate rule live? — Resolved, auto-pick: the raw-go steer, with stencils pointing at it (D2).
- Q4: May a fork background anything? — Resolved, auto-pick: no (D3).
- Q5: Render relative paths into the fork prompt? — Resolved, auto-pick: no; Read and Write need absolute paths, so only the `cd` invitation goes (D4).
- Q6: Do the orch's own agent briefs need the rule? — Resolved, auto-pick: forks inherit the steer; fresh agents get it through a pointer in the orch's brief (D6).
- Q7: Does Master need a guard for an interim notice? — Resolved, auto-pick: yes, one line in loop step 4 (D5).
- Q8: Does Claude Code honour `BASH_MAX_TIMEOUT_MS` above ten minutes? — Open: assumed yes; listed under Open risks, observable in the first Webster run after landing.
- Q9: How do the idle watchdogs see a Merriam waiting on a working fork? — Resolved, auto-pick: claudeengine's activity reading counts the session's fork transcripts and an in-flight fork tool call, the latter for at most the Bash timeout ceiling past the fork's last write (D9).
- Q10: Does a fork that sent an interim notice stay outstanding in shuttle? — Resolved from evidence: no; Claude Code reports it `finished` and the Stop payload lists only running tasks, so D5 states its bounds against a held turn end and a gated arrival.
