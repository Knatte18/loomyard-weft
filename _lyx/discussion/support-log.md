# Support log — webster-parent-merge-midrun

## Interview

The session ran autonomously; every turn below is a self-resolved question, grounded in a read of the code.

1. **Is this the right thing to build, or should mid-run merges be forbidden outright?**
   A parent merge was needed to pick up a deployed fix, so forbidding it moves the wedge instead of removing it.
   Tolerating merge-only movement in the head check and warning at the operator verb covers both halves of the board's fix direction.
2. **What exactly does "merge commits only" admit?**
   A first-parent walk from HEAD that reaches the report's `head_sha` through commits with two or more parents only.
   It rejects a fork that committed past its own report, which is the case the head check exists for.
3. **Which SHA is the batch recorded at after a merge?**
   The report's `head_sha`: `CardSHAs` is the bisect anchor and the post-batch delta is the batch's own work, so the merge commit would attribute parent changes to the card.
   `recover-batch` already records `digest.HeadSHA`, so the two verbs converge.
4. **Does `recover-batch` need the same change?**
   Yes: its head cross-check is a copy of `record-batch`'s and wedges the same way.
5. **What if the merge-in conflicted and is still in progress when Master records?**
   `DoneChecks` and drift would read conflict-marked files, so both verbs refuse until the merge concludes; the batch stays non-terminal and retryable.
6. **Where does the "mid-run" warning fire, and how does it know?**
   In the `lyx fabric merge-in` CLI verb, the path the hub used.
   Detection is a webster-owned predicate over `state.json` and `outcome.yaml`, since only webster may construct its own paths.
   Landing merges through `fabricengine` after Webster is done, so it neither reaches the CLI verb nor matches the predicate.
7. **Warn or refuse?**
   Warn: once `record-batch` tolerates the merge, a refusal would only block a legitimate fix pickup.
8. **What does "loom" warn mean here?**
   The loom-run actor that merged was the hub driving repairs, whose guidance is the `ly-drive` skill; one rule there is the loom-side half.
9. **Does a mid-run merge break anything else?**
   `begin-batch` captures a fresh `StartSHA`, so the next batch is unaffected.
   Integration bisect over earlier `CardSHAs` runs on pre-merge trees and can misattribute; recorded as an open risk and named in the warning.
10. **Testing approach?**
    Hermetic git fixtures in `websterengine` for the head rule, the in-progress refusal, and the end-to-end wedge; predicate unit tests; a hubforge fixture test for the `fabric merge-in` warning.

## Rejected alternatives

- **Refuse `lyx fabric merge-in` while Webster is mid-run.** Blocks the legitimate case that triggered the bug (picking up a deployed fix); with D1 in place the merge is survivable.
- **Accept any HEAD that descends from the report's `head_sha`.** Would also accept a fork's own post-report commits, defeating the head cross-check.
- **Verify each merge's second parent is on the task's recorded parent branch.** `websterengine` does not know the parent branch (fabric does), and the case it would catch — a fork merging after its own report — is narrow; the fork audit does not cover it, since it allows repo-native git, so it stays an open risk that D3's warning makes visible.
- **Record the batch at the post-merge HEAD.** Attributes the parent's changes to the batch's cards in the delta, scope guard, bind checks, drift, and bisect.
- **Put the warning inside `fabricengine.MergeIn`.** Landing's own post-Webster parent merge goes through the engine and would need the predicate to suppress it; the engine would also gain a webster dependency. The CLI verb is the operator path and the right place.
- **Detect mid-run from loom's shed status (current phase is Webster).** Couples fabric to loom's status file layout and misses a standalone-launched Webster run's state; webster's own files are the direct signal.
- **Teach integration bisect about merge commits (insert merge SHAs as bisect candidates).** Not asked for; the warning names the attribution risk. Kept as an open risk.
- **Make `begin-batch` consume an existing OK report instead of refusing.** Blurs the bracket discipline; `record-batch` is the verb that consumes a report, and D1 unblocks it.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Forbid or tolerate mid-run parent merges? — auto-pick: tolerate in the head check, warn at `fabric merge-in` (resolved).
- Q2 Definition of merge-only reconciliation? — auto-pick: first-parent walk, every walked commit has ≥2 parents, must reach report `head_sha` (resolved).
- Q3 SHA recorded for the batch after a merge? — auto-pick: report `head_sha` for `CardSHAs`, delta, drift trigger; tree checks read the merged worktree (resolved).
- Q4 Apply to `recover-batch`? — auto-pick: yes, same shared helper (resolved).
- Q5 Merge in progress at record time? — auto-pick: refuse, batch stays non-terminal (resolved).
- Q6 Where does the warning fire? — auto-pick: `fabriccli` `merge-in` RunE only, never `fabricengine` (resolved).
- Q7 How is "mid-run" detected? — auto-pick: webster-exported predicate; `state.json` present and outcome absent or not `done` (resolved).
- Q8 Warn or refuse at merge-in? — auto-pick: warn; predicate errors degrade to no warning (resolved).
- Q9 Loom-side warning? — auto-pick: one rule in the `ly-drive` skill; no loom driver change (resolved).
- Q10 Check merge second-parent provenance? — auto-pick: no; webster does not know the parent branch, and the fork audit allows repo-native git, so the residual is uncontrolled (resolved, listed as open risk).
- Q11 Change integration bisect? — auto-pick: no; warning names the risk (resolved, listed as open risk).
- Q12 Warning envelope key in fabric's JSON output — auto-pick: a `warnings` string array, the key webster's own CLI envelopes use (resolved).
- Q13 Admit a fast-forward merge-in in D1? — auto-pick: no; telling a fast-forward from a fork's own post-report commits needs the parent branch, which webster does not know (resolved, listed as open risk).
