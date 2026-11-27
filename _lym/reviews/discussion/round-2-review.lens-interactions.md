```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I ran each decision's paths with the others in force and checked the criteria against them.

**Pairs checked**
- D5 × D1, a second landing on the same parent target (one landing entry against two runs): **broken** (F-I1)
- D5 × D8, the catch-up's synchronous PushRun against D5's hold on the pair's records: **broken** (F-I2)
- D5 × D13 × D8, Worktree-Teardown removing the pair, and the composite's `done` write, once the catch-up has pushed the held records: **broken** (follows from F-I2)
- D5 × D3, the landing entry against the ledger entry's removal rules and the all-clear: **broken** (F-I3)
- D5 × D6, batten's Land row after a local landing while origin is unreachable: **broken** (F-I4)
- D3 × D7 × D5, who clears the refused mark, and whether the pusher's Start refuses a refused target: **broken** (F-I5)
- D6 × D3, the wait record of a dead driver against the summary and the all-clear: **broken** (F-I6)
- D5 × D7, a diverged local parent read as "a refusal (D7)": **broken** (F-I7)
- D5 × D6, re-step after `handedOverStop`'s git-transport stop past the merge (held=true path, reconcileVerdict proceeds, merge up to date, push step again): held
- D5 × D6, the owner checks and `syncParent`'s fetch as git-transport, with no parent lock held during the wait: held
- D5 × D2, the push-now mark on the parent target and on the pair target after the parent push lands: held
- D5 × D13, the dobby's landed-pair teardown gated on "parent owes no push": held, absent F-I2
- D1 × D3, the board target's owed entry: held
- D1 × D8, the catch-up kept synchronous and an owed failure starting the pusher: held
- D2 × D12 × D13, `PushUntilLanded` removed from the pusher, and the teardown's own use of it removed by D13: held
- D11 × D13, the pushes-off teardown criterion: held
- D3 × D4, the bar's minutes against waiting runs (a waiting run's pair is also owed): held
- D9 × D5, removal on resume against the merge record: left to the other lenses
- Constraint PATTERN-pair-remote (records before code) × D5's hold: held for the pair pusher; broken by F-I2's synchronous path
- Constraint PATTERN-owner-machine × D5/D6: held for Finalize's own reads

**Findings**

**F-I1 — BLOCKING, design, `_lym/discussion/decision-record.md:113-134` (D5, with D1:43-46)**
D5 gives each parent target one landing entry: "the parent target's record of the owed parent tip". That entry holds one run's records commit and one `done` write. D5:122 says a second landing on an ahead parent leaves "the parent target then owes the newer tip". If that replaces the entry, run A's `done` write is lost and run A's pair hold is released, so A's records reach origin before the parent push. If entries add up instead, the record never says so: one entry per run, each settled when origin's parent holds its tip. Neither D5 nor the D5 second-landing criterion (:319) says which.
Evidence: decision-record.md:113-114, 122, 130-134, 319; landing entry singular throughout.

**F-I2 — BLOCKING, design, `_lym/discussion/decision-record.md:133-135` (D5) against D1:57 and D8:200-204**
D5 promises that "origin never reads the run's records ... whose parent lacks the run's work", because the pair target holds Finalize's records commit. The catch-up stays synchronous (D1:57). `pushOwedForCatchUp` pushes the prime's records and the pair through `pairteardown.PushRun` and never asks the pusher. So a dobby start or wake while a landing push is owed pushes the held records commit, and the code side after it, straight to origin. After that, origin's tracking refs hold both tips, D13's Worktree-Teardown removes the pair "as today", and the composite marks the landed entry done. The `done` mark then reaches origin before the parent push. D5 or D8 has to state that the catch-up honours the landing entry's hold.
Evidence: battencli/catchup.go:265-276; pattern/PATTERN-pair-remote.md:82-84 (the composite marks a landed run done after removal); decision-record.md:57, 133-135, 261-262.

**F-I3 — MEDIUM, consistency, `_lym/discussion/decision-record.md:86` (D3) against D5:130-138**
D3 removes a target's entry "when the push lands", or when the target's worktree is removed or gone. D5 removes the landing entry only after the `done` write, and keeps it while it is refused. The record never says whether the landing entry is part of D3's ledger entry. If it is, D3's rule drops it when the parent push lands and before the `done` write, which loses the write and the pair hold. If it is not, D3's all-clear ("once the ledger is empty") can post while a landing entry still holds a run's records and `done`.
Evidence: decision-record.md:84-86, 95, 113-114, 130-138.

**F-I4 — MEDIUM, design, D5/D6 (`_lym/discussion/decision-record.md:116-119, 153-158`)**
D6 waits out only stops with the transient class git-transport or github-api. After Finalize's local landing returns done, batten's Land row pulls the prime (`fab.Pull`). On an unreachable origin that pull returns an error, and `pullStopOutcome` passes it through, so the row is Stuck with "pull failed: …". That is a blocked stop with no transient class, so it goes to a hand-back or the tackler instead of a wait. The run stops on a transient outage right after the landing, against the Goal's "only a refusal that retrying cannot fix stops anything". The record also never says what the Land pull does while local main is ahead of origin by an owed landing tip.
Evidence: battencli/land.go:107-113, 168-173; battenshed/land.go:265-270; sheddriver/driverules.go:289-300, 305-316.

**F-I5 — MEDIUM, design, `_lym/discussion/decision-record.md:88-90` (D3) against D7:178 and D5:136-138**
D3 says clearing the refused-push record "removes the mark and lets the pusher start again", but names no writer. D7's way forward ends in `lym fabric push`, which clears only the failed-push record on landing (pushclassified.go `landed` → `j.clear()`) and touches no ledger. The dobby's ahead check writes nothing for a marked target, so the mark can outlive the record. The bar's minutes then keep counting and the all-clear never posts. D5 settles a refused landing entry "on the next pusher pass or dobby tick". But `Start` and `Conditions` refuse while `RefusedPushStands`, so only the tick path exists, and D3 is silent on it. The mark should be read from the record (`RefusedPushStands`), or the record should name the writer that clears it.
Evidence: runpusher/start.go:122-125; runpusher/run.go:65-68; decision-record.md:88-90, 136-138, 184.

**F-I6 — LOW, consistency, `_lym/discussion/decision-record.md:156-162` (D6) against D3:92-96**
D6 says a wait record under a free driver lock is a dead driver, reported once as a stop, but names nothing that removes the record. D3 lists every waiting run in the summary, and its all-clear needs "no run is waiting". If D3 reads that stale record as a wait, the all-clear never posts. If the record stays, it counts toward the summary threshold.
Evidence: decision-record.md:92-96, 156, 161-162.

**F-I7 — LOW, consistency, `_lym/discussion/decision-record.md:121` (D5) against D7:178**
D5 calls a local parent diverged from origin's "a refusal (D7)". D7's refusal leaves a failed-push record and a notice and stops the pusher. `syncParent`'s diverged case pushes nothing and is a Stuck today. The record never says whether Finalize now writes a failed-push record for a read, which would also set D3's refused mark on a target that never pushed.
Evidence: landingshed/finalize.go:621-622; decision-record.md:121, 178.

Tool calls:
1. `grep pullStopOutcome` in battencli and the runfsm `BoardDone`/`PastParentMerge` uses: my context lacked what the Land row's pull does with an unreachable origin and how the FSM derives landed.
