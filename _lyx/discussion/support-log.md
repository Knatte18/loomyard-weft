# Support log — creel-stage1-fixes

## Interview

### Turn 1 — scope and order (auto)

The board entry lists eight defects and puts #677 first.
Autonomous mode: no operator answers; each pick is recorded in the ledger.
The reed defects #679 and #680 stay out, as the entry says, and C13's stacked half belongs to `creel-c13-stacked-rewrite`.

### Turn 2 — root causes read from the code (auto)

- #677: `foldJournal` skips the last non-blank line of the newest file when it is invalid, but `repairTail` returns early on a file ending in a newline, so the skipped line stays and becomes a middle line.
- #681: creelcli's `refuse` writes the error envelope to stdout through `output.Err`, which ignores the write error, so nothing reaches stderr.
- #682: the standing stall is cleared only by a probe that reads the pane; a held receiver's probes skip the read, and wait registration seeds a new waiter from the standing stall.
- #683: after a spawn the client connects with a 1, 2, 4, 8, 8 s backoff (23 s) and never checks the spawned pid.
- #684: `apply` returns for a carried record before appending to the event list; live retention writes carried lines without folding them.
- #685: two misses in a row with a 30 s base doubling to 4 min; the second miss waits out the backoff.
- #686: the adapter's `Idle` returns no stall for a pane that shows no input box; expiry is checked only at a probe, and the backoff probe times (15, 23, 31 s) match both observed overruns.
- #678: a plain `reed add --cmd claude` strand has no shuttle startup to accept the folder-trust gate; the test prints no capture on the idle wait.

### Turn 3 — design picks (auto)

Picks D1 to D9 as recorded in the decision record; alternatives in `## Rejected alternatives`.
Diagnosis kept here, not in the record: the probe backoff of 1, 2, 4, 8 and 8 s puts probes at 15, 23 and 31 s, matching the 23.3 s and 31.4 s expiries; the 80 s sweep came from the second miss waiting out a 30 s or longer backoff.

### Turn 4 — lens review

Every lens returned a report.

- board-fit MEDIUM: the smoke helper's parentless `reed add` hits #673 and may draw the pane too short. Fixed: D9 adds `--anchor own-window` and names placement as a second candidate cause.
- board-fit MEDIUM: overlap with `reed-add-parentless-placement` on `SANDBOX-CREEL-SUITE.md`. Fixed: Scope out-list and an open risk keep this task to C4, C6, C8, C10 and C11.
- board-fit LOW: #680 affects a sandbox re-check from a strand pane. Fixed: open risk added.
- board-fit LOW: no conflict with creel stages 2 and 3. No change needed.
- premises MEDIUM and goal-scope MEDIUM (merged): the `--wait-reply` read-mail risk is false, since the sweep covers only unread mail and queued injects, and an orch-role holder has no worker for the nudge. Fixed: D6 drops the worker nudge for a 50 s ceiling plus the 10 s confirm, which covers orch holders; the risk is replaced.
- premises LOW: shuttle already reads no-input-box on the readiness path. Fixed: D7 takes the field from the existing readings.
- premises LOW and pattern-fit BLOCKING (merged): D9 played claude gate keys from creelcli, against PATTERN-shuttle-provider-seam. Fixed: D9 goes through a new engine-neutral `shuttleengine` seam reached via `shuttleengines`, with its bound stated.
- pattern-fit MEDIUM: D4's reap waits on the daemon but logged nothing. Fixed: the exit is logged at Info, and the constraint is rewritten.
- pattern-fit LOW: D8's worker-wait change is missing from the daemon-wakeups update. Fixed: Scope and Constraints name the worker line.
- pattern-fit LOW: D7's field source is unstated. Fixed with the premises LOW above.
- goal-scope MEDIUM: no acceptance line for "the smoke test passes". Fixed in part: the open risk states the operator's by-hand run proves it; no gate can run the llm tier, so no criterion can require it.
- goal-scope LOW: C4 and C8 watch lines are missing. Fixed: added to Scope.
- goal-scope LOW: "#677 first" has no plan effect. Fixed: plan-writer note.
- goal-scope LOW: D2 widens the board's ask. Kept as an auto-mode assumption: every creel refusal shares the path.
- goal-scope LOW: D3 and D6 may be oversized for a low-budget task. D6 fixed by dropping the nudge; D3 kept, since freshness alone does not tell a waiter about a stall that persists.
- scenarios MEDIUM: accepting the trust gate writes the temp path into the operator's Claude config. Fixed: stated and accepted in D9 for a by-hand llm run.
- scenarios MEDIUM: a failed listing must not nudge. Moot once the nudge was dropped; the existing failed-listing rule is now in D6 and the acceptance criteria.
- scenarios MEDIUM: a 10 s confirm window may release a re-added strand's mail. Fixed: named in Open risks, with the bound stated in D6.
- scenarios LOW: D3's "once" is ambiguous for A, B, A. Fixed: once per change, with an acceptance case.
- scenarios LOW: D3's bound tied to a constant D8 changes. Rejected: D8 changes no constant, it only caps a wait.
- scenarios LOW: D4's reconnect and late-death cases. Fixed: the signal is per spawn and ignored after a connect; the no-signal path is in the criteria.
- scenarios LOW: repeated carries. Fixed: D5 states one entry per carry.
- scenarios LOW: D8's criterion was garbled. Fixed: within one second of the TTL.
- scenarios LOW: a closed pipe sends SIGPIPE before any stderr write. Fixed: one clause in D2.
- scenarios LOW: D9's boot wait had no bound. Fixed: bounded by `smokePaneWindow`, waiting for box or gate.
- scenarios LOW: D1's empty-file and blank-tail edges. Fixed: added to the criteria.
- leanness MEDIUM: D3 named a rejected alternative. Fixed: clause cut.
- leanness LOW: D5 restated a rejected alternative. Fixed.
- leanness LOW: D8 and D6 held diagnosis. Fixed: moved to Turn 3.
- leanness LOW: the refusal-way-forward non-constraint. Fixed: cut.
- leanness LOW: auto-mode assumptions repeated the decisions. Fixed: trimmed to the choices.
- leanness LOW: plan-writer notes overlapped `relevant.md`. Fixed: bare pointers cut.
- leanness LOW: the Scope in-list repeats the decisions. Rejected: it gives the module boundary per defect.

## Rejected alternatives

- #677: make the fold refuse a newline-ended torn line. Rejected: the board expects the cut, and a refusal keeps creel down.
- #681: change `internal/output` to report write errors for every module. Rejected: out of the task's scope, and the board asks only for creel.
- #682: print the stale stall marked as past. Rejected: nothing in a past stall is actionable, and the board allows dropping it.
- #682: clear the standing stall on every probe that skips the pane read. Rejected: a draft that persists would flap between journaled and cleared, adding `stalled` records.
- #682: clear the stall only when a worker retires. Rejected: does not cover the observed case, where a hold keeps the worker alive with no pane reads.
- #683: check the pid with a signal-0 probe. Rejected: an unreaped child stays a zombie and reads alive.
- #684: reconstruct the deleted original events from the carried snapshot. Rejected: invents history the journal no longer holds.
- #685: lower the sweep's backoff to a fixed 30 s. Rejected: up to 60 s before the confirm, and no backoff at all while nothing changes.
- #685: a delivery worker that misses its receiver nudges the sweep. Rejected after the lens review: an orch-role holder has no worker, and the nudge adds a second signal path to a low-budget fix.
- #678: have the test import `claudeengine` to accept the gate. Rejected: PATTERN-shuttle-provider-seam keeps startup-gate keys inside `shuttleengine`.
- #685: one miss counts as gone. Rejected: drops the guard against a strand briefly missing from a listing.
- #686: only document that a non-agent pane names no reason. Rejected: the stall is cheap to name and it also names a dialog covering the box.
- #678: switch the smoke test to a shuttle-launched strand, which dismisses the gate. Rejected: the test proves delivery into a hand-added strand.
- #678: pre-seed Claude's trust config for the temp hub. Rejected: ties the test to Claude's config file format, which belongs under the provider seam.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): fix #677 at the repair or the fold? Repair cuts the tail (D1).
- Q2 (auto-pick): fix #681 in `read` only or for every creel refusal? Every creel refusal (D2).
- Q3 (auto-pick): drop or mark a stale stall? Drop it, by a freshness bound of twice the probe ceiling (D3).
- Q4 (auto-pick): how does the client learn its spawned daemon died? An exit signal from a background reap (D4).
- Q5 (auto-pick): how does `log` show a carried message? As a `carried` event entry (D5).
- Q6 (auto-pick): how to bring the sweep within about 60 s? A 10 s confirm interval and a 50 s backoff ceiling (D6), revised from a worker nudge after the lens review.
- Q7 (auto-pick): name a reason or document the reasonless expiry? Name a `no-input-box` stall (D7).
- Q8 (auto-pick): fix the TTL overrun by capping the probe wait at the remaining TTL (D8).
- Q9 (auto-pick): #678's cause taken as the strand placement (#673) or the folder-trust gate, both addressed, with the capture printed on failure (D9); unconfirmed, see the decision record's open risks.
