---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:48
    summary: Decision 5 bound compares fabric push and sync to a commit-gated push although both push unconditionally, and Auto-mode assumption 3 ignores the automatic sync that lyx config runs in the prime
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:38
    summary: Decision 3 states a duration bound for the blocking lock that its own Open risks entry contradicts, so a stalled network push can hold every status transition with no limit
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:28
    summary: Decisions 1 and 2 and the acceptance criteria leave open what the entry returns and what commit-records and the seam report when both sides fail with different error kinds
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:48
    summary: Decision 5 does not say which both-sides path lyx fabric push takes and silently drops the rebase recovery that its current PushCoalesced route performs on a rejection
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:57
    summary: Constraints omit PATTERN-refusal-way-forward and the refusal-spec row for commit-records, whose push-failure message and way forward this task changes
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:52
    summary: The PATTERN-push-both-sides entry text has no exemption for the prime, so review would flag the records-only push that Scope Out keeps for Batten as a violation
  - id: F7
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:70
    summary: Auto-mode assumption 1 and the first Notes bullet restate content held elsewhere or re-derived at Plan time
---

### [MEDIUM] Decision 5's bound does not hold for unconditional pushes

**Location:** `_lyx/discussion/decision-record.md:48` (Decision 5), `:72` (Auto-mode assumption 3), `:21` (Scope Out).
**Issue:**
Decision 5 bounds the prime exposure with "which `lyx fabric commit` already does whenever it lands something; nothing new in kind".
That is true of `Fabric.Commit`, which spawns its both-sides detached push only when `WarpCommitted || WeftCommitted` (`internal/fabricengine/commit.go:169`).
`lyx fabric push` and `lyx fabric sync` push whether or not the call committed anything: `push` calls its push step unconditionally and `sync` spawns its child unconditionally (`internal/fabriccli/weft_verbs.go:230`, `:306`).
Once they carry the code side, a no-op call in the prime publishes the operator's unpushed parent-branch commits, which the commit-gated route does not.
Callers of `sync` are not only explicit: `lyx config` runs `fabriccli.RunCLIIn(cwd, w, []string{"sync"})` after an edit (`internal/configcli/configcli.go:502`), from whatever worktree the operator is in, the prime included.
Auto-mode assumption 3 says Batten "keeps the parent branch out of its automatic pushes", and Interview 4 gives exactly that exposure as the reason Batten stays records-only.
Decision 5 never says why the same exposure is acceptable through `sync`, and the record never mentions the automatic caller.
**Fix:**
State the real bound in Decision 5: both verbs push unconditionally, so in the prime they publish any unpushed parent-branch commits even when the call committed nothing, including through the `sync` that `lyx config` runs; what bounds it is that the push is a plain, rebase-free push of committed commits on the checked-out branch, never a force.
Reword assumption 3 so it names only Batten's per-transition seam and points at Decision 5 for the verbs.

### [MEDIUM] The blocking lock's stated bound is contradicted by Open risks

**Location:** `_lyx/discussion/decision-record.md:38` (Decision 3), `:77` (Open risks).
**Issue:**
Decision 3 bounds a status push's wait at "at most one push's duration per holder", then Open risks says a hung push child holds back every transition "until git gives up".
The two cannot both be the bound.
`lock.AcquireWriteLock` blocks with no timeout (`internal/lock/lock.go:21`), and `internal/gitexec` sets no command timeout, so a stalled network holds the lock for as long as git's transport waits.
A holder under the new entry is also not one push: it is both sides, each with a possible fetch and a retry push.
The status seam's contract is that a push failure warns and the run continues (an offline laptop must not kill an autonomous run), and acceptance criterion 9 says the dispositions are unchanged.
A wait with no limit turns that into a stall of the run's own transition, which is a changed disposition.
Decision 3 introduces that edge and does not bound it.
**Fix:**
Decision 3: the status-side acquisition (the new entry and `PushAnchored`) waits for the lock for a bounded, injectable time and, when it expires, returns an error the seam warns about ("next transition will catch up"); the detached child keeps its blocking acquire.
Replace the "one push's duration" claim with the actual holder cost, and rewrite the Open risks lock entry to the residual risk: the holder can still run long, but no status transition waits past the bound.
Add an acceptance criterion that a status push against a held lock warns after the bound and the run continues.

### [MEDIUM] Both-sides failure reporting is left open

**Location:** `_lyx/discussion/decision-record.md:28` (Decision 1), `:34` (Decision 2), `:80` (Acceptance criteria).
**Issue:**
Decision 1 returns `gitrepo.ErrPushRejected` unwrapped when either side stays rejected, returns every other push error as an error, and "reports both failures", but the entry returns one `error`.
When the code side is rejected and the records side fails with a network error, the unwrapped sentinel hides the second failure and any wrapper breaks the "unwrapped" claim.
The record does not say how both failures are carried or how a caller tells which side failed; loomcli may not name sides, so the text has to come from `fabricengine`.
Decision 2 and the acceptance criteria say only that `commit-records` yields its error envelope and the seam warns.
The current envelope text says the commit "was not pushed" (`internal/loomcli/commitrecords.go:42`), which is false when only the code side failed and the records side was pushed.
**Fix:**
Decision 1: a single failed side returns its error as today (a rejection bare); when both fail the entry returns `errors.Join` of the two, each message naming which side failed, so `errors.Is(err, gitrepo.ErrPushRejected)` holds if either side was rejected.
Decision 2: the `commit-records` envelope and the seam warning carry the joined text, and the envelope wording no longer claims the records were not pushed.
Acceptance criteria: one scenario where the two sides fail with different error kinds, asserting the returned error identifies both and the records side was still pushed.

### [MEDIUM] Decision 5 leaves the push path open and drops a recovery

**Location:** `_lyx/discussion/decision-record.md:48` (Decision 5), `:95` (Notes).
**Issue:**
Decision 5 says `push`'s normal mode goes "through the both-sides lock path instead of `Fabric.PushWeft`".
Two entries fit: `CoalescePushBothAt`, which swallows a rejection as a warning and exits ok, and the new entry, which surfaces it; the Notes say the new entry must surface rejections, but Decision 5 does not pick.
`Fabric.PushWeft` runs `PushCoalesced`, which on a rejection pulls with rebase and retries (`internal/fabricengine/weftgit.go:319`), so `lyx fabric push` can today resolve a weft remote that moved ahead.
After this change it is rebase-free, so that recovery disappears for `lyx fabric push`, and the way forward `commit-records` names ("`lyx fabric push` pushes the landed commit") no longer resolves a rejection.
The record does not state the loss or what the verb does on a rejection.
**Fix:**
Decision 5: `push` normal mode calls the new entry, so a rejection after the one retry is an error envelope and not a swallowed warning.
State that `lyx fabric push` no longer rebases the records branch, bounded by Decision 4: it retries only when the remote tip is already contained locally and otherwise leaves both branches alone.
Add one acceptance criterion for `lyx fabric push` against a real divergence: error envelope, no rewrite.

### [LOW] Refusal pattern and spec row missing from Constraints

**Location:** `_lyx/discussion/decision-record.md:57` (Constraints); `contracts/specs/refusal-spec.md:212`.
**Issue:**
`PATTERN-refusal-way-forward` requires a refusal's message to name the way forward and its spec row to land in the same commit as the code.
The row "commit-records: not pushed" names `lyx fabric push` as the way forward, and Decisions 1, 2 and 5 change both when it fires and what that way forward does.
The Constraints list neither the pattern nor the spec row, and the "Docs land in the same commit" bullet names none of them.
**Fix:**
Add `PATTERN-refusal-way-forward` to Constraints: the `commit-records` push-failure message and its `contracts/specs/refusal-spec.md` row change together with the code, and the way forward is correct for a real divergence as well as a transient failure.

### [LOW] PATTERN entry text has no prime exemption

**Location:** `_lyx/discussion/decision-record.md:52` (Decision 6).
**Issue:**
The entry reads "Pushing a task pair's records side from Go: the code branch goes with it ... pushing is never left to an agent", and says it is enforced by review.
Scope Out keeps Batten's seam, which runs in the prime, pushing the records side only.
A reviewer applying the entry as written would flag that as a violation, and the reason it is allowed (the prime's code branch is the operator's parent branch) is only in Scope Out and the support log.
**Fix:**
Word the entry so the exemption is part of it: in a task pair the code branch goes with the records side; the prime's per-transition push stays records-only.

### [NIT] Duplicated assumption and a perishable call-site list

**Location:** `_lyx/discussion/decision-record.md:70` (Auto-mode assumption 1), `:94` (Notes, first bullet).
**Issue:**
Assumption 1 opens by restating the #418 cause already given in Decision 3's rationale, and its retry clause repeats Decision 4's rationale; only "the drive report was not inspected" is new.
The first Notes bullet lists the files and functions that hold today's pushes.
The writer's stencil tells it not to gather call-site lists, `Plan-Write` derives them with its own quarry lookups, and a rename silently falsifies the list.
No requirement or constraint lives in either, so under the completeness-before-leanness test nothing needs extracting first.
**Fix:**
Cut assumption 1 to the unverified fact: the cause is inferred from the code and the drive report was not inspected.
Delete the first Notes bullet; the second carries a requirement already in Decision 1 and stays, and the third stays.

## Focus checks without findings

Decision 4 against the real-divergence criterion is coherent.
The retry fires only when the fetched remote tip is an ancestor of, or equal to, local HEAD, and a real divergence stops after the one fetch, as criterion 5 says.
The second push is a plain push, so if the remote moves between the fetch and the retry, git's own fast-forward check rejects it: no case lets the retry push over remote commits.
The rejection classifier in `gitrepo.PushRebaseFree` matches `rejected`, so a remote ref-lock failure (`[remote rejected]`) does reach the retry, as the rationale claims.
