---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D7
    summary: D7's way forward for the reported state (local rewound, remote holding the run's abandoned commits) is a merge that pulls those commits back and can conflict with the redone work, and it contradicts D7's own premise that the guard discards only commits the local branch holds
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D7
    summary: D7 runs the remote update before the local reset's ownership and dirtiness checks, and states neither what the verb reports when those checks then refuse nor what a non-lease remote failure (network, auth) leaves and names
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D6
    summary: D6 rests on a failed, dead or stuck batch having no fork running, but a dead batch keeps its strand alive by design and may still be working
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4
    summary: D4 and D6 name recover-batch as the way forward for every batch, but recover-batch refuses a failed batch carrying uncheckable paths with run --fresh, and D4's recover-batch check does not say which refusal comes first
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3
    summary: D3's stated bound does not bound the residue it names, because delete-not-done passes a Delete target that is not found, so a never-held or wrongly removed target passes both checks
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D1-D4
    summary: D1 to D4 scope the dispatch checks as cards not yet completed, but the existing scoping drops every begun card and has a forthcoming subset, and D4 does not say which later cards it compares
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: the D6 acceptance criterion covers failed and stuck batches only, while the Decision accepts dead as well
  - id: F8
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D7
    summary: D7 leaves open a remote tip that is an ancestor of the reset commit, where the update would push forward instead of rewrite
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D1
    summary: D1 claims to mirror deriveEdges exactly, but deriveEdges draws edges only between cards of different batches while D1 compares every pair of cards
  - id: F10
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D10
    summary: D10 removes a guard without stating what it lets through, and its rationale describes the assertion as checking the fork's model while the code asserts Master's own role model
  - id: F11
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md Open risks
    summary: the first Open risks bullet restates D2's Bound sentence
---

Focus item on the lease (D7) is settled with no finding: the Decision ties the lease to the tip the ancestry check ran against, makes a moved tip refuse before any mutation, and the refusal names the merge way forward.
Focus item on D3 and the Rework-Plan-Gate is settled except for F5: `ValidateRework` runs `ValidateFormat` with no completed cards, so a missing Delete target is still refused there, as D3 states.
Focus item on D6's terminal statuses is settled by F3 and F7: `BatchState.Status` holds `done`, `stuck`, `dead`, `failed`, and D6 accepts three and refuses the other plus non-terminal.

### [MEDIUM] D7's merge way forward does not hold for the state the bug reports

**Location:** `_lyx/discussion/decision-record.md` D7.
**Issue:** In #404's state an earlier reset already rewound the local branch, and the remote task branch holds the run's own abandoned commits.
D7 refuses that state and names "merge `origin/<task-branch>` into the task branch, then re-run the reset".
That merge brings the abandoned commits back into the local branch, can conflict with the batches the run redid since, and exists only to make them reachable so the reset can discard them again.
D7's own Rationale says the guard "can discard only commits the local branch already holds, which are the run's own", and in this state the local branch does not hold them.
The Decision therefore never states how the reported state converges without a conflict-prone detour, and it does not separate the run's abandoned commits from someone else's pushed ones, which is what the guard exists to protect.
**Fix:** Decide the way forward for a remote tip holding commits the local HEAD lacks.
State in D7 whether the merge is the accepted route and who resolves a conflict, or whether lyx can recognise the run's own abandoned commits from what the run recorded, and what bounds that recognition.
Add the chosen behavior to the acceptance criteria as the #404 state converging.

### [MEDIUM] D7's order of remote update and local checks leaves the failure outcome open

**Location:** `_lyx/discussion/decision-record.md` D7.
**Issue:** The remote update runs before `ResetPairWarp`, whose ownership and dirtiness checks (a dirty tracked path the run did not write, a merge in progress, an off-branch HEAD) are the common local refusals.
After the remote has been rewound, such a refusal leaves the remote at the reset commit and the local branch ahead.
D7 says a re-run converges, but the verb's existing message for that failure reads `webster: reset --to X refused: ...`, which tells the operator nothing changed, and the refusal needs a way forward that includes the remote having moved (`PATTERN-refusal-way-forward`).
D7 also states only the lease refusal for the remote step, not a network or auth failure of the update itself, which leaves nothing changed but needs a named re-run.
**Fix:** State in D7 whether the remote update runs after the local checks pass or stays first, and why.
State the message a local refusal after a completed remote update carries, and the refusal for an operational remote failure, each with its way forward and its `refusal-spec.md` row.

### [MEDIUM] D6 asserts that a dead batch has no fork running

**Location:** `_lyx/discussion/decision-record.md` D6 Rationale.
**Issue:** D6 says "a failed, dead or stuck batch has no fork running".
`internal/websterengine/recoverbatch.go` keeps the strand of a dead-classified batch alive ("dead-classified strand may still be working") and `recoverSpawn` stops the prior live strand before it spawns the recovery.
The restamp is safe for `dead`, but because recover-batch reclaims the strand, not because none is running.
**Fix:** Replace the premise with the true one: the batch is terminal, and recover-batch stops any live prior strand before rendering from the current plan.

### [MEDIUM] The recover-batch way forward does not hold for a batch with uncheckable paths

**Location:** `_lyx/discussion/decision-record.md` D4 and D6.
**Issue:** D4's way forward ends in `recover-batch NN`, and D4 makes `recover-batch` run the D2 check before spawning for any batch, not only one that failed on `delete-not-done`.
`RecoverSpawnOrAttach` refuses a failed batch with `Uncheckable` entries and sends it to `run --fresh`.
A batch that failed on an audit finding with uncheckable paths and whose card also deletes a still-referenced target gets D4's refusal naming `recover-batch`, then recover-batch's own refusal naming `run --fresh`.
D6 acknowledges this refusal for the restamp but not for D4, and D4 does not order the two refusals.
**Fix:** State in D4 the order of the two refusals at recover-batch time, and that the way forward names `run --fresh` instead of `recover-batch` when the batch's record carries uncheckable paths.

### [MEDIUM] D3's bound does not bound the residue it names

**Location:** `_lyx/discussion/decision-record.md` D3 Bound.
**Issue:** D3 says a Delete target missing for any reason, including one the tree never held, passes with the advisory alone, and bounds that with "`record-batch`'s `delete-not-done` check still runs ... so a card whose wanted state does not hold is not recorded done".
`DoneChecks` reads `not_found` as the wanted state for a Delete target, so for a target that was never there the check passes too, and the advisory is the only remaining signal.
The real bound is that the Plan-Gate saw the target exist before the run started, so only a target the run removed, or a mid-run plan edit, can reach this state.
**Fix:** State the actual bound and drop the claim that `delete-not-done` bounds the residue.

### [LOW] The card-scope phrase differs from the existing dispatch scoping

**Location:** `_lyx/discussion/decision-record.md` D1, D2, D3, D4.
**Issue:** D1 says "cards not yet completed (the existing dispatch scoping)", D3 says "not-yet-completed card", D4 says "later cards not yet completed, as at dispatch".
`DispatchScope` in `internal/websterengine/beginbatch.go` drops every card whose batch has a record (begun), and keeps a `forthcoming` subset of non-terminal begun cards for the Create and Rename checks.
D4 also adds the batch's own begun cards as the deleting side without saying whether a later card of a begun batch is compared.
**Fix:** Use one phrase for the scope, the cards of batches with no record, and state in D4 which later cards it compares and that the batch's own cards are the deleting side.

### [LOW] The D6 acceptance criterion omits dead batches

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria.
**Issue:** The criterion names "a failed or stuck batch", while D6 accepts `failed`, `dead` and `stuck`.
**Fix:** Name `dead` in the criterion, as a row in the same scenario.

### [LOW] D7 leaves a remote tip behind the reset commit open

**Location:** `_lyx/discussion/decision-record.md` D7.
**Issue:** The update is skipped only when the remote tip equals the reset commit.
A remote tip that is an ancestor of the reset commit passes the ancestry check, and "update to the reset commit" would then push forward, which is Publish's and fabric's job and not a rewrite the Bound describes.
**Fix:** State that the update is skipped when the remote tip is an ancestor of the reset commit.

### [LOW] D1 mirrors `deriveEdges` across batches it does not compare

**Location:** `_lyx/discussion/decision-record.md` D1.
**Issue:** `deriveEdges` adds edges only between cards of different batches, so a `Uses` that names a later card's target in the same batch produces no edge and no reorder.
D1 compares every pair of cards and claims the rule is exactly `deriveEdges`'.
**Fix:** Either state that the check applies across batches only, or state that same-batch pairs are refused as well and why that stricter rule is acceptable.

### [LOW] D10 states no bound for the removed assertion

**Location:** `_lyx/discussion/decision-record.md` D10.
**Issue:** The removed injection asserts Master's own role model onto Master's pane, so after the change a pane that drifted from its configured model is no longer switched back.
The Rationale says "the fork inherits Merriam's model, so `begin-batch` has no model to check", which describes a different check.
**Fix:** State what the removal lets through, a Master pane on a model other than its role's, and what bounds it.

### [NIT] An Open risks bullet repeats D2's Bound

**Location:** `_lyx/discussion/decision-record.md` Open risks.
**Issue:** The first bullet restates the false-positive sentence of D2's Bound.
**Fix:** Delete the bullet, or reduce D2's Bound to the pointer-free statement and keep the cost in Open risks.
