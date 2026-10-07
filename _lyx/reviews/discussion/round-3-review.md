---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md Decision 5 and Open risks
    summary: Decision 5 bounds what the prime's code-side push publishes but not how it fails; a behind or diverged parent branch now turns lyx fabric push into an error envelope though the verb only pushed the records side before
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md Decisions 1-3 and Acceptance criteria
    summary: A lock-wait expiry returns an error that names no side, contradicting "every error it returns names the failed side", and the commit-records envelope for an expiry is neither decided nor covered by a criterion
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Decision 2 and Constraints
    summary: The envelope message must vary with the failed side and with rejection versus other failure, yet loomcli may name no side and the record does not say where the rejection discrimination lives
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md Decision 2
    summary: The class of the new commit-records rejection row is not named although refusal-spec rows carry a class that fixes the disposition
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Constraints, PATTERN-no-denied-recovery entry
    summary: The constraint names two of the denied forms while the pattern's closed list holds more, so a reader could treat the other forms as allowed
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: The single-side criterion covers only a code-side failure and does not assert errors.Is for one rejected side or the envelope when only the records side fails
  - id: F7
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md Decision 1 and Open risks
    summary: Decision 1 points at Decision 3 for the unbounded choice that Decision 5 makes, and the last Open risks entry repeats Decision 3's closing paragraph
---

### [MEDIUM] Decision 5 does not bound the failure the prime's code-side push now adds

**Location:** `_lyx/discussion/decision-record.md`, Decision 5 and Open risks.
**Issue:** `gitrepo.PushRebaseFree` runs a plain `git push` with no `HasUnpushed` gate, and `rebaseRetryTriggers` maps `non-fast-forward`, `rejected` and `fetch first` to `ErrPushRejected`.
A parent branch that is merely behind its remote, or diverged, or protected, is therefore rejected, and Decision 4's retry cannot absorb it because the remote tip is not contained locally.
Decision 5 turns a rejection that stays into an error envelope for `lyx fabric push`, and the prime is not distinguishable in the layout (support-log Rejected alternatives), so `lyx fabric push` in the prime now fails on the operator's own branch after pushing the records side.
Today it pushes the weft only and never touches that branch.
Decision 5's `Bound:` paragraph states what is published and that it is never a force, but says nothing about this new failure, and Open risks lists no matching entry.
The attack-surface question asks what the new routing edge can let through and what bounds it; the failure it adds is the part left unstated.
**Fix:** In Decision 5's `Bound:` paragraph, state that in the prime a rejected parent-branch push returns the error envelope after the records side was still pushed, that nothing local is rewritten, and that the way forward is Decision 2's rejection row.
Add an Open risks entry for the prime's parent branch being behind or protected, and extend the `lyx fabric push` real-divergence criterion to name the code side as the failing side.

### [MEDIUM] A lock-wait expiry has no side and no decided `commit-records` outcome

**Location:** `_lyx/discussion/decision-record.md`, Decision 1, Decision 2, Decision 3 and Acceptance criteria.
**Issue:** Decision 1 says "Every error it returns names the failed side and that side's worktree path".
Decision 3 makes the entry return an error on lock-wait expiry, before either side is attempted, so that error has no failed side.
Decision 2's envelope text "names the failed side" and "does not say the records were unpushed unless the records side failed" has no reading for an expiry, where the records side is in fact unpushed.
`commit-records` is one of the bounded callers (Decision 3), yet the only expiry criterion is "A status push against a lock held past the wait bound warns and the run continues", which covers the seam and not the envelope.
**Fix:** In Decision 1, scope the side-naming sentence to push failures and add that a lock-wait expiry returns an error naming the push lock and no side.
In Decision 2, say that an expiry falls under the transient row and that its envelope makes no claim about a side.
Add an acceptance criterion: `commit-records` against a lock held past the wait bound returns the transient envelope.

### [LOW] The envelope text conflicts with the vocabulary constraint and leaves the discrimination site open

**Location:** `_lyx/discussion/decision-record.md`, Decision 2 and Constraints (`PATTERN-fabric-vocabulary`).
**Issue:** Decision 2 requires the `commit-records` message to vary: it names the failed side, omits "records were unpushed" unless the records side failed, and picks between the rejection and the transient way forward.
The Constraints say `loomcli` names no side, so `loomcli` cannot choose wording by side, and non-owner `.go` files cannot spell the sides at all.
The record does not say where the rejection-versus-other split is made, though it needs `errors.Is(err, gitrepo.ErrPushRejected)` on the entry's error, and `loomcli` does not import `gitrepo` today.
**Fix:** In Decision 2, state that the envelope's fixed text is side-neutral, that only the entry's own error text carries side names and worktree paths, and that the rejection split is made on `errors.Is(err, gitrepo.ErrPushRejected)` in `loomcli`, or by a neutral predicate exported from `fabricengine`.
Reword the single-side criterion to match (see F6).

### [LOW] The new rejection row has no named class

**Location:** `_lyx/discussion/decision-record.md`, Decision 2.
**Issue:** Every row in `contracts/specs/refusal-spec.md` carries a class (correctness halt, policy guard, transient) that fixes its disposition.
The existing `commit-records: not pushed` row is transient, the Publish push-rejected rows it is modelled on are correctness halts, and Decision 2 splits the row without saying which class the rejection row takes.
**Fix:** In Decision 2, name the class of each of the two rows: both transient, since the verb halts without mutating state and the message names the retry, with the merge as the named action on the rejection row.

### [LOW] The denied-forms constraint lists only part of the closed list

**Location:** `_lyx/discussion/decision-record.md`, Constraints.
**Issue:** The entry says "no refusal or warning names `git push --force` or `git reset --hard`".
`PATTERN-no-denied-recovery` holds a closed list that also covers `git push -f` and `rm -rf`, and says `agent-permission-setup` updates it.
Enumerating a subset in the record goes stale and invites the plan writer to treat the unlisted forms as allowed.
**Fix:** Reword the constraint to "no refusal or warning names a form on `PATTERN-no-denied-recovery`'s denied list".

### [LOW] The single-side criterion is one-sided

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria.
**Issue:** The criterion for a one-side failure covers the code side only.
It does not assert that `errors.Is(err, gitrepo.ErrPushRejected)` holds for one rejected side, which Decision 1 states and the row choice depends on, and it does not cover the records side failing while the code side was pushed, where Decision 2 requires the envelope to say the records were unpushed.
**Fix:** Extend the criterion to a failure of either side: the error names that side and its worktree, `errors.Is` holds when that side was rejected, and the envelope's claim about the records side is true in each case.

### [NIT] A wrong pointer in Decision 1 and a repeated risk

**Location:** `_lyx/discussion/decision-record.md`, Decision 1 and Open risks.
**Issue:** Decision 1 says the lock wait is "bounded or unbounded (Decision 3)", but the unbounded choice is Decision 5's.
The last Open risks entry restates the closing sentences of Decision 3 (a status push that stalls in git stalls its own transition), which the second entry already covers.
**Fix:** Point Decision 1 at Decisions 3 and 5, and fold the last Open risks entry into the one before it.
