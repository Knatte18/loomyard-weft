---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:112
    summary: D7 plain-merge way forward leads a re-run of the reset to discard the very commits the merge integrated, so it destroys someone else's work and is conflict-prone, and nothing says who resolves a conflict
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:109
    summary: D7 leaves open which refusal wins when the remote-ancestry refusal and a local gate refusal both apply, and what reset does offline or with no remote configured
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:74
    summary: D4 batch-failed reason and D6 rebaseline name recover-batch for a batch whose record carries uncheckable paths, which recover-batch refuses with run --fresh, so the named way forward does not work the first time
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:97
    summary: D6 accepts a restamp for a dead batch whose strand may still be working on the old card, and its Bound does not state that exposure or what bounds it
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:147
    summary: D10 rationale and Bound describe a per-batch switch and a drifted-pane check, but the code compares a recorded model with the run's own launch model and never fires in the shipped flow
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md:36
    summary: Support log says reset can tell the run's abandoned commits from someone else's, while D7 says lyx cannot tell the two cases apart
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:193
    summary: Acceptance criteria miss D7 network or auth failure and the no-remote and SkipPush skips, D6 dead batch with a live strand, and D10 removal of the launch-time baseline
  - id: F8
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:41
    summary: D1 claims Webster runs gated plans in card order, which holds only when every batch is a contiguous card range, and the Batcher contract does not promise that
  - id: F9
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:35
    summary: D1 says the check compares only unbegun cards but does not say which card its finding is attributed to, and dispatch scoping drops pure findings by that card alone
---

### [BLOCKING] D7's plain-merge way forward discards what it merged

**Location:** `_lyx/discussion/decision-record.md:112` (and `:111`, `:125`).
**Issue:**
D7 says the merge "is only a hand-off to the reset: the re-run discards what it merged".
The refusal then offers a plain merge "when the listed commits may hold someone else's work".
That path ends with the re-run rewinding the local branch and force-updating the remote to the reset commit, which drops the merged commits from both.
The Bound ("only when every commit it drops is reachable from the local HEAD") is met trivially by any merge, so it bounds nothing for someone else's work.
The plain merge is also the conflict-prone form (the remote holds commits touching the same files as the redone batches), and no Decision says who resolves the conflict or how.
The Open risks entry names discarding someone else's pushed commits as the danger, yet the decided way forward produces it whenever the operator follows the plain-merge form.
A rejected-alternatives line cannot carry this fix: the plain form is a Decision, so the record must state either that it is intended to discard or drop it.
**Fix:**
Rewrite the refusal's way forward as one path that converges and one that stops.
The listed commits are the run's own abandoned ones: `git merge --strategy ours origin/<task-branch>` in the task worktree, then re-run the reset.
The listed commits may be someone else's: do not re-run the reset, because the reset discards them; the refusal says so and names the commits' authors' branch as the place to keep them, leaving the rewind to the operator.
Restate the Bound: the remote update drops only commits reachable from the local HEAD, and lyx puts them there only through a merge the operator ran after reading the listed commits.

### [MEDIUM] D7 leaves refusal order and offline reset undecided

**Location:** `_lyx/discussion/decision-record.md:109`, `:114`, `:116`.
**Issue:**
The ancestry refusal runs "before any mutation", while the remote update runs inside `ResetPairWarp` after its containment, ownership and dirtiness checks.
So a diverged remote and a dirty tracked path can both apply, and the record does not say which refusal the operator sees first.
Following the ancestry refusal's merge into a dirty worktree and then meeting the dirty refusal is a second round-trip the goal ("works the first time") rules out.
D7 also makes `webster reset` need the network: the fetch and the lease update refuse offline, and the record is silent on a repo with no `origin` configured.
Before D7 the reset was local-only.
The network and authentication failure is covered by one clause (`:116`); the no-remote-configured case, which is a different state from "the remote has no such branch" (`:118`), is not.
**Fix:**
State the order (gate checks first, so the first refusal is local and cheap, then fetch and ancestry), or state the opposite and why.
State that a repo with no remote configured skips the remote update like `SkipPush`, and that an unreachable remote refuses with the re-run way forward the record already names.
Each distinct refusal gets its `refusal-spec.md` row, as the Constraints already require.

### [MEDIUM] D4's and D6's way forward names `recover-batch` for a batch it refuses

**Location:** `_lyx/discussion/decision-record.md:74`, `:77`, `:102`.
**Issue:**
`BatchFailedError.Error` (`internal/websterengine/batchfail.go`) ends with `way forward: lyx webster recover-batch NN` and does not look at the record's `Uncheckable` list.
`RecoverSpawnOrAttach` refuses a failed batch with uncheckable paths (`ErrRecoveryNeedsFresh`) and names `run --fresh`.
D4 orders `recover-batch`'s two refusals correctly (`:77`) but only for `recover-batch`'s own refusal.
The batch-failed reason D4 changes (`:74`) is rendered at failure time and still says `rebaseline --card NN`, then `recover-batch NN`, whatever the record holds.
A `delete-not-done` failure that also carries uncheckable paths therefore gets a way forward whose last step is refused.
D6 has the same hole from the other side: `rebaseline --card` succeeds on that batch (`:102` says only that `recover-batch`'s own refusal then names the way forward), so the operator spends a rebaseline on a batch that cannot be re-driven.
**Fix:**
D4: the batch-failed reason names the `run --fresh` steps when the batch's record lists uncheckable paths, and D4's `rebaseline`/`recover-batch` way forward otherwise.
D6: `rebaseline --card` on a failed batch whose record lists uncheckable paths refuses with the same `run --fresh` way forward, and the Bound says so.
Add one acceptance row for the uncheckable case.

### [MEDIUM] D6's restamp of a dead batch states no bound on a live strand

**Location:** `_lyx/discussion/decision-record.md:97`, `:101`, `:104`.
**Issue:**
`awaitTerminal` in `recoverbatch.go` keeps a dead-classified strand alive ("dead-classified strand may still be working"), and `recoverSpawn` stops a live prior strand only when `recover-batch` runs.
So between the restamp and `recover-batch`, a dead batch's strand can still be working on the old card and can write a late report.
That is the exposure the record cites for refusing a non-terminal batch (support log "D6, accepting an edit to a begun non-terminal batch: its fork may be in flight on the old card"), accepted for `dead` without a stated reason.
The premise that `recover-batch` stops a live prior strand and renders from the current plan holds (`recoverSpawn` calls `removeStrandIfLive` and re-hashes the cards at spawn).
What is missing is the Decision stating what the restamp lets through (a window with the old-card strand live) and what bounds it (the next `recover-batch` stops the strand and archives a late report, `refuseRecoveringDoneReport`'s dead and failed exception).
**Fix:**
Add that sentence to D6's Bound, and name `recover-batch` as the step that ends the window in the dead case's way forward.

### [MEDIUM] D10 describes a check the code does not perform

**Location:** `_lyx/discussion/decision-record.md:144`, `:147`, `:149`.
**Issue:**
D10's rationale calls the assertion a "per-batch switch of Master's pane".
`BeginBatch` compares `State.AssertedModel` with `RoleMaster`'s resolved model, and `Run` sets `AssertedModel` to that same model at every entry (`runlevel.go:698`).
The package doc already says the injection is dormant in the shipped flow: the comparison never finds a divergence without manual state tampering.
The Bound's claim that removal lets "a Master pane that drifted from its role's model, by an operator's `/model`" stay on it is therefore wrong in two ways: the code never read the live pane, so it never corrected such drift, and a recorded-versus-configured divergence cannot arise in the shipped flow.
What removal does let through is only the manual-tampering case.
The record also ties D10 to batch-validation speed (`support-log.md` interview item 4), but a dormant comparison costs nothing, so the speed claim has no ground in this Decision.
Last, the removal list omits the launch-time baseline write in `runlevel.go`, which goes with `State.AssertedModel`.
**Fix:**
Restate the rationale as removal of a dormant seam: the model is set at spawn and `begin-batch` never changes it.
Restate the Bound: the removal lets through nothing in the shipped flow, since the check never read the pane; a hand-edited `assertedModel` no longer causes an injection.
Drop the speed implication, and add the `Run` baseline write to what is removed.

### [LOW] Support log contradicts D7 on telling the commits apart

**Location:** `_lyx/discussion/support-log.md:36` against `_lyx/discussion/decision-record.md:113`.
**Issue:**
The rejected-alternative line says "Publish cannot tell the run's abandoned commits from someone else's; reset can, from the pre-reset HEAD".
D7 says "Lyx cannot tell the two cases apart" and rests the guard on the operator's judgment.
Both can hold only if reset can tell when the local HEAD still holds the commits and cannot after an earlier reset rewound it, which neither file says.
**Fix:**
Make the support-log line say that reset can tell only while the local HEAD still holds the commits, and cannot in the #404 state.

### [LOW] Acceptance criteria miss decided behaviors

**Location:** `_lyx/discussion/decision-record.md:193`, `:192`, `:196`.
**Issue:**
D7 decides a network or authentication failure refuses and changes nothing, and that the remote update is skipped under `SkipPush` and with no such remote branch.
The reset criterion covers the diverged remote, the moved tip, the local gate refusal, the failed checkout rewrite and the ancestor tip, but not these.
D6's criterion names `failed, dead or stuck`, so every terminal status the Decision accepts appears, but it has no row for a dead batch with a live strand (F4) or one with uncheckable paths (F3).
D10's criterion does not cover the removed `Run` baseline write (F5).
**Fix:**
Add the rows, as additions to the existing reset and rebaseline criteria rather than separate items.

### [LOW] D1's card-order guarantee assumes contiguous batches

**Location:** `_lyx/discussion/decision-record.md:41`.
**Issue:**
D1's rationale says that once no `Uses` edge points backward, `SequenceBatches` returns the declared order unchanged, so Webster runs gated plans in card order.
`deriveEdges` works on batches and `batchIdentity` numbers a batch by its first card.
A grouping batchifier that groups non-adjacent cards (the `Batcher` interface promises no contiguity) can give a batch-level cycle from card-forward edges, which `SequenceBatches` condenses.
The claim holds for the identity batcher, the only one there is, and D1's same-batch sentence (`:36`) already speaks for a grouping one.
**Fix:**
State the assumption: the guarantee holds when every batch is a contiguous run of card numbers, as the identity batcher's is, and nothing else is claimed.

### [LOW] D1 does not say which card its finding is attributed to

**Location:** `_lyx/discussion/decision-record.md:35`, `:37`.
**Issue:**
D1 says the check "compares only unbegun cards" at dispatch.
In `planglyph.ValidateDispatch` the pure `ValidateFormat` pass runs over the whole plan and a finding is dropped only when its `Card` field is a begun card.
So D1's scoping comes entirely from which card the finding carries, and D1 names both cards in the text without choosing the attributed one.
A pair with one begun and one unbegun card is kept or dropped depending on that choice.
D4 compares a begun deleting card against later unbegun cards (`:75`), so both pairings occur in the design.
**Fix:**
State the attributed card (the consumer, whose `Uses` names the later target), and that a pair with a begun card on either side is dropped.

## Focus verdicts

- **D7 abandoned-commit state** — the refusal converges without a conflict only through `git merge --strategy ours`; the plain merge is conflict-prone and discards what it merged, and nobody is named to resolve a conflict (F1). Acceptance criteria cover the state (`:193`) but not the someone-else's-work branch.
- **D7 order of steps** — the remote update sits after the gate checks and before the rewrite, so a local refusal leaves the remote unchanged (stated, `:114`); a failed rewrite after the remote moved is stated with its message (`:117`); an ancestor or equal remote tip is skipped (`:118`). Open: refusal order and offline or no-remote (F2), and the acceptance gap (F7).
- **D6 premise** — holds against `recoverbatch.go`: `recoverSpawn` stops a live prior strand, and `awaitTerminal` keeps a dead strand alive. The acceptance criterion names `failed`, `dead` and `stuck`, all three terminal statuses the Decision accepts. The window with a live dead-batch strand is unbounded (F4).
- **D4 and D6 versus uncheckable paths** — not settled; the batch-failed reason names `recover-batch` unconditionally (F3). The order of `recover-batch`'s two refusals is stated at `:77`.
- **D3 Bound** — not a defect. `DoneChecks` reads `not_found` as the wanted state for a Delete target (`stillExists := r.Status != quarry.StatusNotFound`), as the Bound says. A target the tree never held is bounded only by the Plan-Gate having seen it exist before the run, which the Bound states.
- **D1 to D4 card scope** — the phrase "batches with no record" matches `DispatchScope` (begun is every recorded batch; forthcoming is its non-terminal subset). D1's match rule mirrors `deriveEdges`; its scope is wider by design (same-batch pairs), and it says so. Two gaps: which card carries the finding (F9) and the contiguity assumption (F8). D4 does say which later cards it compares (later unbegun cards).
- **D10** — the Decision does state a Bound, but it misdescribes the check (F5).
