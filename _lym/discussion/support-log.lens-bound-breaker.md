```yaml
lens: bound-breaker
outcome: reported
```

Lens bound-breaker: I tested each claim, bound and guarantee in the pawl-orch-landing decision record against concrete paths. Two findings are BLOCKING, five MEDIUM and three LOW.

**Findings**

1. **BLOCKING — D4 (and D8, D3 refusal 6, D7): a pair forked from the beam inherits the tracked beam pull-request record.**
   - Counterexample: beam B runs `lym beam pr open`, which records pull request #10 (head B, base parent) under B's `_lym/landing/`. The operator then makes a nested beam C under B; `beamMembers` lists "open tasks and beam projects" under a beam. C's pair, forked from B, inherits the record at the same landing path. C's Finalize (D8, "a record stands") then squash-merges #10 on GitHub, which lands B into B's parent. C's `lym beam ready` judges #10's merge state, and C's dobby posts D7 notices about B's pull request.
   - Evidence: `internal/pawlengine/doc.go` says the ready record "names the beam it flags, since a pair forked from the beam inherits it", and `ReadyFor` reads a record naming another slug as no flag. D4 has no such guard.
   - Fix: the record names its beam slug, or Head must equal the reader's branch, and a mismatch reads as no record.

2. **BLOCKING — D2 bound and D3 refusal 2 vs the entry's "From `lym beam ready` on … no member lands into it": a member filed after ready runs and lands into the beam.**
   - Counterexample: the orch runs `lym beam ready` on an idle beam, then files member M. The status is still Beam-Await, so `RunHolds` is false and the board accepts M. In the dobby's tick, `startReady` (loop.go:588) runs before `landBeam` (loop.go:600), so M starts before any land request is written. `landBeam` then sees work, writes no request, and the beam waits through M's whole run. M lands into the beam after ready, after the merge-state check.
   - The record's bound says such a member "is caught by the re-check, never landed over" and calls the window narrow. The re-check only fires if the request is written first, and the window is M's whole run, not about a second. The entry's guarantee is broken.
   - Fix: while the ready record stands, the dobby starts no member and the board refuses new members, or ready writes the hold at once.

3. **MEDIUM — D3 refusal 6 reads the head GitHub holds, not the head Finalize will push.**
   - Counterexample: a merge-in task lands into the beam locally, and the orch runs `lym beam ready` without `lym beam pr update`. Refusal 3 passes, because the local head contains the parent. Refusal 6 judges the stale pull-request head: it can read `clean`, or `behind`/`dirty`, and the way forward names `pr status`, not `pr update`.
   - When it reads `clean`, Finalize pushes a new head, so required checks restart, and a "dismiss stale reviews" rule drops the approval. `mergeOnGitHub` retries pending for only `prMergeRetryWindow` (one minute), then halts, past the point of no return.
   - Fix: refuse when the pull request's head SHA differs from the beam's head, naming `lym beam pr update`. Name `behind` among the causes.

4. **MEDIUM — D3 refusal 6 and D6 open/status: a recorded pull request that was closed or merged on GitHub has no named cause and no way forward.**
   - Counterexample (issue #57's case): the pull request is closed on GitHub during Beam-Await. `mergeable_now` is false, but the cause list (review, check, conflict, draft, unknown) has no "closed". `pr open` reports `already` because a record stands, and no reopen verb exists. The only way out is `pr close` then `pr open`, which the record never states.
   - For a pull request merged by hand on GitHub, `pr close` would ask GitHub to close a merged pull request, which fails. The beam cannot reach ready.
   - Fix: state the closed and merged causes and their ways forward. `pr open` could reopen a closed recorded pull request.

5. **MEDIUM — Goal, D3 and D10 claim "no way back to Beam-Await", but `lym loom goto` gives one.**
   - `CheckGotoTarget` admits any earlier row, so `lym loom goto Beam-Await` from a halted Finalize is accepted.
   - On entry, Beam-Await finds the board status naming Finalize with no land request, reads it as an interrupted landing, lifts the hold and tells the orch to `lym beam land`. Members can then join and land. The ready record still stands, so the dobby lands the beam again with none of D3's refusals rerun.
   - Fix: state goto as the hand escape and what it lets through, or refuse a pawl goto to Beam-Await past ready.

6. **MEDIUM — D13 "when the new binary takes it" is unproven for a run whose old-binary driver is still alive.**
   - Counterexample: a pawl run waits at PR-Review (an in-process wait with no timeout) when the binary updates. Unless something restarts that driver process, the old code goes on: an approval leads into the old Finalize, and the retired-row move never fires.
   - I found a reexec on binary change only for the dobby loop (`internal/dobbyengine/loop.go`), not for the run drivers.
   - Fix: the record should name what ends or reexecs a live old driver, or bound D13 to runs whose driver is not live.

7. **MEDIUM — D8 ordering: removing the record vs the parent pull.**
   - If the record were removed before the parent pull succeeds, a crash re-enters Finalize with no record. Under beam.yaml `on`, D8's last case then lands again by a local squash merge and push, on top of the GitHub merge.
   - The record lists "pulls the parent, and removes the record" but never states that the order is load-bearing. `mergedButNotPulledReason` depends on the record still standing at re-entry.
   - Fix: state that the record is removed only after the parent-side landing completes.

8. **LOW — D3 refusal 3: a parent with no upstream** (a local hub, or push skip set) has no "upstream tip". The record does not say whether the local tip alone is judged or the beam is refused.

9. **LOW — D13: a run halted at a retired row needs two resumes.** `lym loom resume` steps the run, the move halts it again at Beam-Await, and a second resume follows. The notice should say so, or the move should apply on resume without a fresh halt.

10. **LOW — D5 and D6 "not at Beam-Await" is ambiguous for a run halted at Beam-Await**, the state D13 leaves runs in. The record should state that a halted status at that row counts as at Beam-Await.

**Claims attacked**

- D1, rows and registry: held.
- D2, Finalize as the hold name: held.
- D2, window bound: broken (finding 2).
- D3, "checked before the point of no return": broken (findings 3 and 4).
- D3, "no verb removes the ready record": held, but goto bypasses ready's finality (finding 5).
- D4, one record shared by the verbs and Finalize: broken by inheritance (finding 1).
- D6, `open` is idempotent: broken for a closed pull request (finding 4).
- D6, `mergeable_now` set: held. `unstable` with only non-required checks failing is mergeable.
- D7, at most one repeated notice on a fresh machine: held.
- D8, required pull request with no record: held. `resolveWithParentRule` re-asks with the GitHub rule on a no.
- D8, crash safety: needs the ordering stated (finding 7).
- D9, unedited text changes nothing for a task: held. The risk is already in Open risks.
- D11, a supervisor re-invokes: held. `sheddriver/driverules.go` and `shedverbs` read the policies.
- D12, the dobby is never started on a held beam: held for the paths I traced (Beam-Await's StartDobby runs only at the Beam-Await status).
- D13, coverage of in-flight runs: weak (findings 6 and 9).
- Constraint on flock and close-on-exec: held.
- PATTERN-pull-request-mode rule text: by the parent's count, 282 characters reduced by a 4-character edit, so within the 280 cap.

**Tool calls**

- Grep for users of `InterruptPolicyFor`/`InterruptPolicyReinvoke`, `CheckGotoTarget`'s rules, binary-change reexec, and the dobby's start-vs-ready reads: my context lacked who acts on the interrupt policy, whether goto may move back past Finalize, and whether the dobby starts members while a ready record stands.
- Grep of the order of `startReady` and `landBeam` in the dobby's tick: needed to confirm that a member starts before the land request in the same tick.
