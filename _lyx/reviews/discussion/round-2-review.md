---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:66
    summary: D3 rests on the false premise that Fabric holds paths and no location, so it leaves open where the Commit route learns it runs in the prime, though Commit already resolves a Location from its warp path
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:69
    summary: D3 states a prime-name resolution failure only for the push and sync verbs, and not what Fabric.Commit's spawn does on that failure, so the Commit route's guard can fail open and publish the parent branch
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:50
    summary: D2 equates the probe's absent answer with an empty adopt tip, but Add skips the probe under SkipGit or SkipPush while step 11 still pushes, so rollback could delete a pre-existing origin branch and the Decision states no bound
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:55
    summary: D2 says a refused or failed remote deletion is recorded in rec, which contradicts PATTERN-mutation-record (success-only) and the Constraints bullet that says the deletion records through the existing success-only entry
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:125
    summary: The mutation-record Constraints bullet files D4's renames under verb-specific fields outside the fixed key set, but the side= trace detail D4 renames is a value inside the mutations entries
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:149
    summary: Acceptance criteria cover neither a mixed remote-lines plus bare-failed stderr being retried, nor an exhausted step 11 rolling back, nor the Commit route in the prime when prime resolution fails, nor a skipped probe
---

### [MEDIUM] D3's "Fabric holds paths and no location" is false

**Location:** `_lyx/discussion/decision-record.md:66` (D3), against `internal/fabricengine/commit.go` (`Fabric.Commit`).
**Issue:** D3 says "`Fabric` holds paths and no location, so the Plan picks where the `Commit` route learns that it runs in the prime."
`Fabric.Commit` already calls `lyxcwd.ResolveWorktree(f.warpPath)` and holds the resulting `*lyxcwd.Location` before it classifies paths, and `PrimeName` takes exactly that type.
The premise is false, and the choice D3 hands to the Plan is already determinate: the route that ends in `spawnDetachedPushFn(f.warpPath, f.weftPath)` can apply `PrimeName(l) == l.WorktreeName` to the location it already resolved.
The focus question was how a route that holds only paths learns it runs in the prime; the answer is that `Commit` does not hold only paths, and the Decision should say so rather than defer.
The other two routes are fine: `PushPairAnchored` takes `l`, and the `sync` verb has `l` in scope.
**Fix:** Replace the sentence with a statement that `Fabric.Commit` decides from the `Location` it already resolves from its warp path, and that the empty code-side path is passed to `spawnDetachedPushFn` there.

### [MEDIUM] D3 leaves the Commit route's behaviour on a prime-name failure unstated

**Location:** `_lyx/discussion/decision-record.md:69` (D3), acceptance criterion at line 156.
**Issue:** "A prime-name resolution failure fails the verb before it commits or pushes anything" names a verb, and the acceptance criterion names `fabric push` and `sync`.
`Fabric.Commit` is also called by every other `Commit` caller, and its spawn is fire-and-forget (`_ = spawnDetachedPushFn(...)`) after the commit has landed.
For that route the Decision does not say whether a resolution failure errors before any lock or commit (a new refusal for every `Commit` caller), or is swallowed and pushes both sides (the guard fails open and re-opens the leak the task closes), or drops the code side (fail closed).
The Decision introduces a guard on this route and gives it no stated bound on the failure path.
**Fix:** State the `Commit` behaviour: on a resolution failure `Commit` returns the error before taking any lock, like its other pre-flight refusals, so no commit lands and no push spawns.
Add an acceptance criterion for the `Commit` route.

### [MEDIUM] D2's "absent" probe answer is not proven when the probe is skipped

**Location:** `_lyx/discussion/decision-record.md:50` (D2), against `internal/fabricengine/add.go` (probe at the pre-flight, step 11 push).
**Issue:** D2 deletes the origin branch only when "the pre-flight probe found no such branch on origin", and says `Add` "carries the probe's absent answer".
`Add` runs `probeWarpLeftover` only inside `if !opts.SkipPush && !opts.SkipGit`; otherwise `warpAdoptTip` stays `""`, which is the same value as an absent branch.
Step 11's `git push -u origin <warpBranch>` is not gated by those options, so under `WEFT_SKIP_PUSH` or `WEFT_SKIP_GIT` a pre-existing origin branch at or behind `HEAD` is pushed forward without ever being probed, and a later rollback would pass D2's deletion condition and, with the lease at the commit step 11 pushed, delete it.
That is the pre-existing-branch deletion the Decision says its guard excludes, with no bound stated for the case where the guard's input was never computed.
**Fix:** State that the deletion requires the probe to have run and answered absent; a skipped probe is not an answer, so the origin branch is left and only the local copy is deleted.
Add an acceptance criterion for it.

### [MEDIUM] D2 says a refusal is "recorded in `rec`", against the success-only record

**Location:** `_lyx/discussion/decision-record.md:55` (D2) and line 124 (Constraints).
**Issue:** D2 ends "Any refusal or failure is logged with `logger.Warn`, as today, and recorded in `rec`."
`PATTERN-mutation-record` says an executor appends its primitive only after it observably changed state, and the Constraints bullet says the remote deletion records "through `deleteRemoteBranch`'s existing success-only entry".
A refused or failed deletion changed nothing, so a plan writer who follows D2 appends an entry for a refused attempt, which is the reading the Constraints bullet exists to rule out.
**Fix:** Drop "and recorded in `rec`" from D2: a refusal or failure is logged and returned, and only a deletion that happened is recorded.

### [LOW] The mutation-record bullet misfiles D4's trace-detail rename

**Location:** `_lyx/discussion/decision-record.md:125`.
**Issue:** The bullet says D4's renamed fields are "verb-specific fields outside that set".
Two of D4's renames are not fields: `side=warp` and `side=weft` come from `refDetail` in `mutation.go` and sit in the `detail` value of entries inside the fixed `mutations` array.
A plan writer reading "the pattern's `mutations` and `partial` keys are untouched" can take the detail rename as forbidden.
**Fix:** Say the `side=` detail is a value inside `mutations` entries, which the pattern's fixed key set does not constrain, while the entry kinds and the keys stay as they are.

### [LOW] Acceptance criteria miss the cases the Decisions add

**Location:** `_lyx/discussion/decision-record.md:149`.
**Issue:** D1 states that `remote:` lines alongside a bare `(failed)` ref line are retried, but the criteria only cover non-`failed` reasons with `remote:` lines (line 151) and a bare `(failed)` (line 149).
D2 states what happens when step 11 exhausts its attempts (nothing landed: refused and logged; landed anyway: deleted), but no criterion covers an exhausted step 11.
The `Commit` route's failure behaviour and the skipped-probe case (F2, F3) have none either.
**Fix:** Add a criterion for each: a stderr mixing `remote:` lines with a bare `(failed)` ref line is retried; an exhausted step 11 rolls back with the lease refusing when nothing landed; the `Commit` route in the prime on a resolution failure; a rollback under a skipped probe leaves origin untouched.
