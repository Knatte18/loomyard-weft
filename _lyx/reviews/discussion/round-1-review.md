---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md, D3 and Scope
    summary: D3 skips the prime's code side only in PushPairAnchored and the sync verb's spawn, but Fabric.Commit also spawns a detached push of both sides whenever a commit lands, so the prime still publishes main and the lyx config acceptance criterion fails
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md, D2
    summary: D2 equates not adopted (no warpAdoptTip) with this Add created the remote ref, but probeWarpLeftover also returns empty for a pre-existing origin branch at or behind HEAD, so the rollback would delete a branch that existed before the call
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md, Constraints PATTERN-mutation-record bullet
    summary: The bullet says the push retries land in Add's Mutations record, which contradicts PATTERN-mutation-record (append only after an observable change) since a refused attempt changes nothing
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, Constraints PATTERN-mutation-record bullet against D4
    summary: The bullet says the envelope key set is unchanged apart from code_push_skipped while D4 renames four envelope keys, because the two sections use envelope key for different things, and a plan writer can read the bullet as forbidding D4
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md, D1 Bound and D2
    summary: D1's Bound rests on the unverified premise that a bare failed rejection updates nothing, and D2 deletes the origin warp branch only after a successful step 11 push, so a step 11 that exhausts its retries on an ambiguous remote state leaves the branch behind
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md, D1
    summary: D1's classifier joins two tests (reason exactly failed, and no hook message) without saying how a stderr with remote lines plus a failed rejection line is classified, or which refs and which line it matches on
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, Acceptance criteria and D2
    summary: No acceptance criterion covers D3's prime-name failure, D4's code_worktree JSON key and records sibling wording, or D5's neutral lock-dir helper, and D2 names an exception line in PATTERN-fabric-git that concerns the archive tag and not origin branches
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, Auto-mode assumptions and Open risks
    summary: The hard-cut assumption says nothing consumes the renamed trace and keys, while the driver stencil reads side=warp from persisted run traces, and the stranded trace open risk states no bound or owner of the handling
  - id: F9
    severity: NIT
    class: scope
    location: _lyx/discussion/decision-record.md, D1 and D2 Rationale, Auto-mode assumptions, Open risks
    summary: Incident narrative from issue 433, a pointer to the support log the plan writer never reads, and an open risk that restates D2's Bound belong in support-log.md or nowhere
---

### [BLOCKING] D3 leaves the prime's `Fabric.Commit` push route open

**Location:** `_lyx/discussion/decision-record.md`, D3, Scope In bullet 2, Acceptance criteria bullet 6.
**Issue:**
D3 changes two routes: `PushPairAnchored` (the `push` verb) and the `sync` verb's own `spawnPush` call.
`internal/fabricengine/commit.go` has a third.
`Fabric.Commit` ends with `spawnDetachedPushFn(f.warpPath, f.weftPath)` whenever a side committed, and `SpawnDetachedPush` runs `lyx fabric --warp-path … --weft-path … push`, whose bypass branch calls `CoalescePushBothAt` on both paths.
So in the prime, `lyx config --set` runs `fabric sync`, `fab.Commit` commits the records file, spawns a child that pushes both sides, and `main` is published.
The `sync` verb's later `spawnPush` with an empty code path does not undo that.
`lyx fabric commit` does the same without any `sync`.
D3's Goal ("never publishes unpushed `main` commits as a side effect of `lyx fabric push`, `lyx fabric sync` or the `sync` that `lyx config` runs") and acceptance criterion 6 (`lyx config --set` leaves origin's `main` unchanged) therefore cannot hold under D3 as written.
This is a missing decision, not a call-site enumeration gap: D3's premise is that the two named routes are the ones that hand the code side to a push, and the code shows a third that does so on every landed commit.
D3's rationale line "`fab.Commit` commits the records side" is true but does not address the push `Commit` spawns.
**Fix:**
D3 states that the prime's code side is dropped at every route by which fabric hands it to a push, `Fabric.Commit`'s spawn included, and names the bound (a push removed, never added, in the prime only).
It also states how that route learns it runs in the prime, since `Fabric` holds paths and no `lyxcwd.Location`.
Scope In gains the route, and an acceptance criterion pins `lyx fabric commit` in the prime.

### [BLOCKING] D2 treats "not adopted" as "created by this call"

**Location:** `_lyx/discussion/decision-record.md`, D2 Decision (the two-bullet condition) and Bound.
**Issue:**
D2 deletes the origin warp branch when "the warp branch was forked, not adopted (no `warpAdoptTip`)" and step 11 succeeded, and its Bound says this touches "only the one warp ref the same `Add` call created and pushed".
`probeWarpLeftover` returns an empty adopt tip in two cases: the origin branch is absent, or the pair is not live and the origin tip equals or is an ancestor of the HEAD the new branch forks from.
In the second case the origin branch pre-exists, step 11 pushes it forward (or finds it equal), and `Add` records `branch_created` for the local branch and `branch_pushed`.
D2's lease is the tip step 11 pushed, which that forward push makes the current origin tip, so the lease passes and the rollback deletes a branch that existed before the call.
Under the default empty `branch_prefix` that branch is a bare slug an operator may own.
The driver stencil's stranded-branch rule already warns that a `branch_pushed` entry alone never qualifies, "it is recorded whenever an existing branch is pushed forward".
The Bound is therefore false for this case, and the Decision does not say what the guard lets through.
**Fix:**
D2's condition is "the pre-flight probe found no branch on origin", not "no `warpAdoptTip`", and the Decision says the probe's absent answer is what `Add` carries to the rollback.
An acceptance criterion pins the case: a rollback after a push-forward of a pre-existing origin branch leaves origin's branch untouched.

### [MEDIUM] Constraints say push retries land in the mutation record

**Location:** `_lyx/discussion/decision-record.md`, Constraints, `PATTERN-mutation-record` bullet.
**Issue:**
`pattern/PATTERN-mutation-record.md` says "An executor appends its primitive only after it observably changed state".
A refused `(failed)` attempt changes nothing, so recording it as a mutation would make `Add`'s record claim a change that did not happen.
Only the accepted push records `branch_pushed`, once, as it does today.
The bullet as written would send a plan writer to add per-attempt entries, a violation of the pattern the bullet cites.
The remote deletion is correct: `deleteRemoteBranch` records on success only.
**Fix:**
Reword the bullet: the rollback's remote deletion records through `deleteRemoteBranch`'s existing success-only entry, and a push retry adds no entry of its own, so the one accepted push records `branch_pushed` as before.

### [LOW] The envelope-key wording in Constraints and D4 disagrees

**Location:** `_lyx/discussion/decision-record.md`, Constraints `PATTERN-mutation-record` bullet; D4's table row for `warp_branch_deleted` and its three siblings.
**Issue:**
In `PATTERN-mutation-record` the fixed envelope key set is `mutations` and `partial`, and D4's four `*_branch_*` keys and D3's `code_push_skipped` are verb fields outside it.
The bullet instead says the set "is unchanged apart from D3's `code_push_skipped` field, which is a verb field, not an envelope key", which both treats the field as an exception to the set and denies it belongs to the set.
D4 then calls the `remove` verb's fields "envelope keys" (as `internal/fabriccli/fabric.go` and `removefields_test.go` also do).
The two statements agree in substance, since the pattern's two keys are untouched, but a plan writer who takes the bullet's "key set" to cover `remove`'s fields reads D4's renames as forbidden.
`removefields_test.go` pins an "always-present key set" for `remove`, which adds to the confusion.
The plan writer would follow D4, because it is the specific and later statement, but the bullet should not leave the question open.
**Fix:**
Reword the bullet to say the pattern's `mutations` and `partial` keys are untouched, and that the verb-specific fields D3 adds and D4 renames sit outside that set.

### [LOW] D1's Bound rests on an unverified premise, and D2's trigger skips an ambiguous step 11

**Location:** `_lyx/discussion/decision-record.md`, D1 Bound; D2 second bullet of the deletion condition.
**Issue:**
The Bound says the retry repeats only a push that left the remote ref unchanged, "a `(failed)` rejection updates nothing".
Nothing in the record or the support log establishes that GitHub's bare `(failed)` is atomic.
The retry is safe whichever is true, because it re-pushes the same commit to the same ref without force, so the second push is a no-op if the first landed; the Bound should rest on that.
The premise matters elsewhere: if step 11 exhausts its attempts with the ref in fact updated, D2's "step 11 push succeeded" condition is false, no origin deletion runs, and the branch survives, which is the #433 failure the Goal names.
**Fix:**
Restate D1's Bound as idempotence of a non-forced push of one commit to one ref.
State in D2 which way an exhausted step 11 resolves: either the deletion also runs when step 11 was attempted and the probe found the ref absent (the lease then refuses harmlessly when the ref is not at the pushed tip), or the leftover is accepted and named in Open risks.

### [LOW] D1's classifier does not settle a mixed stderr

**Location:** `_lyx/discussion/decision-record.md`, D1 Decision, first paragraph.
**Issue:**
The classifier is "the remote-rejected form whose reason is exactly `failed`, with no hook message".
Those are two tests, and the Decision does not say which governs when stderr holds `remote:` lines and a `[remote rejected] <ref> (failed)` line, nor whether the match is per ref line of a multi-ref push (`Add` pushes one ref) or over the whole stderr.
The exact-reason test alone already separates `(pre-receive hook declined)`, so the "no hook message" clause either is redundant or overrides the reason test.
**Fix:**
Pin one rule, for example: retry when the stderr contains a `! [remote rejected] <ref> -> <ref> (failed)` line for the pushed ref, whatever other lines accompany it; or retry only when no `remote:` line is present.
The plan writer then has a single classifier to implement and test.

### [LOW] Acceptance criteria miss behaviours the Decisions introduce

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria; D2's last line.
**Issue:**
Decisions with no criterion:
- D3: a prime-name resolution failure fails the verb before anything is pushed; `sync`'s envelope has no `push_spawned` entry for the code side.
- D4: the result field `WarpWorktree` becomes `CodeWorktree` with the JSON key `warp_worktree` becoming `code_worktree` (Prune, Reconcile, Status), and the refusal wording "records sibling".
- D5: the neutral `.weft` lock-directory helper that `internal/hubgeom` reaches.
The converse holds: no criterion asserts something no Decision states.
Separately, D2 says `PATTERN-fabric-git.md`'s "rolled-back `Add` exception gains this sentence", but the exception on that line is about skipping the archive tag, and the "no origin branch is ever touched" rule it weakens lives in `rollbackAdd`'s doc, which Scope already lists.
**Fix:**
Add one criterion per bullet above, as a row or assertion of an existing criterion where it fits.
Name the right place for D2's pattern sentence (a new line in `PATTERN-fabric-git.md`'s teardown section, or drop the pattern edit and keep the `rollbackAdd` doc).

### [LOW] The hard-cut assumption and the stranded-trace risk do not fit together

**Location:** `_lyx/discussion/decision-record.md`, Auto-mode assumptions bullet 5; Open risks bullet 2; D4's last paragraph.
**Issue:**
The assumption accepts the hard cut "because nothing outside this repository consumes" the renamed variables, keys and trace detail.
The consumer that matters is inside the repository and reads persisted data: `contracts/stencils/shed/shed-template-driver.md`'s stranded-branch rule parses `side=…` from a run's trace file, and a trace written before deploy still says `side=warp`.
D4 changes the stencil to `side=code` and Open risks bullet 2 describes the consequence, but neither says who handles it or what bounds it.
The stencil's own rule ("any other `side` … fails the rule, and you escalate") supplies the bound: the driver escalates and a human removes the branch.
Open risks bullet 1 (`batcher-v2`) does state its handling, a mechanical rename at landing.
**Fix:**
State in Open risks bullet 2 that the driver's stranded-branch rule escalates on a `side` it does not recognise, so the stranded branch is removed by an operator, and cite the same remedy Scope Out bullet 2 names for branches left by earlier binaries.
Narrow bullet 5 to "nothing consumes them live".

### [NIT] Narrative and pointers that belong in the support log

**Location:** `_lyx/discussion/decision-record.md`, D1 Rationale, D2 Rationale, Auto-mode assumptions bullet 1, Open risks bullet 4.
**Issue:**
The completeness-before-leanness test: the requirement the plan writer needs from each is already in a Decision or Bound.
- D1's "GitHub answered three simultaneous pushes to the same weft repo with one bare `(failed)`" and D2's "which is what the hub had to clear by hand in #433" are incident narrative.
  The classifier's shape is already in D1's Decision.
- Auto-mode assumptions bullet 1 points at "the support log's question ledger", a file the plan writer never reads.
- Open risks bullet 4 restates D2's Bound (the lease keeps it from deleting another actor's push) and adds only the consequence that a real rejection now deletes the branch pushed seconds earlier.
**Fix:**
Cut the two narrative clauses and the pointer; the support log already holds the interview and ledger.
Keep Open risks bullet 4's consequence in one clause, or fold it into D2's Bound and delete the bullet.
