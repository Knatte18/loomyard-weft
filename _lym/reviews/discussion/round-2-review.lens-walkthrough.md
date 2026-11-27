```yaml
lens: walkthrough
outcome: reported
```

Task: walk each decision's mechanism end to end under the walkthrough lens and report the findings.

**Decisions walked**
- D1, plain `git commit` on a pair's code branch past the last transition's named tip: purpose not shown to be met. Today's ahead read does not count that commit, and the record never says the pair push carries it (W5).
- D2, a pusher that is down to a 60 s interval hits one transport failure, then a second pusher process starts: purpose met. The interval in force and the last attempt are kept in the ledger.
- D3, a pair entry the dobby's ahead check wrote, after which the D8 catch-up's synchronous push lands: purpose not met. The entry stays, the bar keeps counting and the all-clear never posts (W3).
- D4, `lym fabric sync-status` run by the bar's `#()` job: purpose met.
- D5, landing A on the prime's `main`, where origin answers the owner checks and pushes are owed: purpose partly met.
  - Broken: landing B on the same parent (W1).
  - Broken: the Land row's prime pull during the outage (W2).
  - Broken: batten's own record in the prime's records (W4).
  - Broken: the order of the pull-request check (W6).
- D6, a `git-transport` stop at Finalize with a repair count above three: purpose met for the wait. The halt-note writer for the stop the wait ends on is muddled (W8).
- D7, a hand commit H on GitHub, then a local commit, then a rejected push: purpose met. H is reachable from no local branch or reflog, so the push is refused naming H.
- D8, the catch-up's push lock is held by a running pusher: purpose met. `Start` finds that pusher's own lock taken, starts none, and the running pusher finishes the push.
- D9, the #242 case on Finalize's first lock-free pass: purpose at risk. The base is read from a merge record that this merge has not written yet (W7).
- D10, D11, D12: no mechanism to run; nothing found.
- D13, Worktree-Teardown right after a landing whose push is owed: purpose met, if W1 and W4 hold. The row ends done, then the dobby removes the pair once the parent target settles.

**Findings**

W1. MEDIUM, design, `_lym/discussion/decision-record.md:122` (D5)
- Summary: a second landing on the same parent overwrites the first run's landing entry.
- Walk: landing A records its entry, which holds A's owed tip, A's `done` write and the hold on A's records. Landing B lands on the ahead parent, and "the parent target then owes the newer tip".
- D5 defines one landing entry per parent target, holding "the run's records commit". So either B's entry replaces A's, and A's `done` write and A's records hold are lost or released early, or entries stack, which the record never defines.
- Evidence: decision-record.md:114, :122, :130-134; finalize.go:619-620.

W2. MEDIUM, design, `_lym/discussion/decision-record.md:5` (Goal) and :146 (D5)
- Summary: batten's Land row still stops on an outage after a local landing.
- Walk: Finalize returns done with the parent push owed, then batten's Land row runs `fab.Pull` on the prime. With origin unreachable it returns Stuck "pull failed: …", which is a blocked stop, not a transient one, so D6 never waits it out.
- This contradicts the Goal's "only a refusal that retrying cannot fix stops anything", and D5 does not cover the row.
- Evidence: battenshed/land.go:265-270; battencli/land.go Pull seam.

W3. MEDIUM, design, `_lym/discussion/decision-record.md:86` (D3)
- Summary: a ledger entry outlives a target that a synchronous push made level.
- Walk: the dobby's ahead check writes an entry. Then a push outside the pusher lands: D8's catch-up `PushRun`, PR-Review's push, cancel's records push, or `lym fabric push` after a refusal.
- D3 removes an entry only when the pusher's push lands, the worktree goes or a tick finds the worktree gone. A pusher that finds its target level exits without pushing.
- So the entry stays: the bar's minutes keep counting, the summary lists a branch that is pushed, and the all-clear never posts. The same gap leaves a refused mark standing after its record was cleared by hand.
- Evidence: decision-record.md:85-90; runpusher/run.go:134-139; battencli/catchup.go:265-275.

W4. MEDIUM, design, `_lym/discussion/decision-record.md:135` (D5)
- Summary: batten's run record reaches origin before the parent push, so D5's "origin never reads the run's records … whose parent lacks the run's work" does not hold.
- Walk: after Finalize, batten moves to Land, then Worktree-Teardown, then done. Those commits go to the prime's records, whose pusher has no hold, so they reach origin while the parent push is owed.
- The run reader then reads `battenAtLanding` as `PastParentMerge` true for a parent that lacks the work.
- Evidence: pairteardown/runread.go:204-208, :269-271; decision-record.md:133-135.

W5. LOW, design, `_lym/discussion/decision-record.md:49` (D1)
- Summary: D1's plain-commit owed rule does not match what the code reads or pushes today.
- Walk: a plain commit past the named tip, with no new records commit.
- Today's `RunCommitsAhead` counts the code side as ahead only while origin lacks the named tip, so this commit reads as not ahead. D1 redefines "owed" but never says the pair push sends the code branch's head past the named tip.
- If the push stops at the named tip, the target stays owed forever.
- I could not confirm which tip the code-side push sends.
- Evidence: fabricengine/pusherreads.go:111-145; decision-record.md:305.

W6. LOW, consistency, `_lym/discussion/decision-record.md:143` (D5)
- Summary: the pull-request check is placed before the step that produces its input.
- D5 has a landing "that needs GitHub" check for an owed parent push "before it takes the parent lock". But Finalize decides whether a landing goes through a pull request in `pullRequestLanding`, which runs only after `guard.ensure` takes the lock.
- Evidence: finalize.go:407-425.

W7. MEDIUM, design, `_lym/discussion/decision-record.md:210` (D9)
- Summary: D9's base comes from a merge record that does not yet describe this merge.
- Walk: in the pass that merges in, `mergeParent` runs the after-merge clean check before it calls `WriteMergeRecord`. So D9's removal, right after the merge-in, finds the previous record: parent-align's, an earlier merge-in's, or none.
- With no earlier record the removal does not run, and the #242 halt comes back. The base this pass has is `before.taskHead`, held in memory.
- Evidence: finalize.go:1349-1390; landingshed/parentalign.go:454.

W8. LOW, consistency, `_lym/discussion/decision-record.md:157-158` (D6)
- Summary: D6 has the driver write a halt note "when it stops waiting on a stop that is no longer network-class".
- That new stop comes from a step that already writes its own note, `loomAfterStep` for a blocked stop or another failed class. So the driver's note is either a duplicate or the wrong writer.
- Evidence: loomcli/halt.go:150-170.

**Tool calls**
1. Grepped the `WriteMergeRecord` callers and read pusherreads.go:141-152. The context lacked the other writer of the merge record and the end of the code-side ahead read.
2. Grepped coalesce.go and pushclassified.go for the code tip the pair push sends, and read parentalign.go:440-456. The context lacked the push tip and parent-align's write of the merge record.
