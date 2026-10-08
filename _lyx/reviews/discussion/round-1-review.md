---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3
    summary: The low-speed abort D3 relies on for #458 fails with curl's Operation too slow text, which IsTransportFailure's phrase list does not match, so that path is not a transport failure
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2
    summary: gc.auto=0 alone does not stop auto-maintenance repacks; git runs maintenance run --auto, whose task set follows maintenance.strategy, so maintenance.auto=false is missing
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D1
    summary: D1 leaves the helper's lock discipline open on a non-reentrant RWMutex - exclusive or shared lock, goGit resolved before locking, and no nested Repo read inside a wrapped closure
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D3
    summary: D3's bound does not state what a timed-out step then does; the shed driver's one transient re-step makes the worst case two deadlines then hand-back, and the Decision should say so
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: No criterion checks D2's pickup of gc.auto=0 on an existing hub at its next removal, pull joining the classified subcommands, or a remote command carrying the low-speed environment
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D6
    summary: D6's new gc PATTERN entry does not state its carve-out from PATTERN-fabric-destruction-chokepoint, though housekeeping deletes unreachable objects outside destroy.go
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md Q6
    summary: Q6 calls the #447 reuse expectation an open tension in the record, but the record states it as a settled assumption and Open risks omits that the operator's expected reuse stays unavailable
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md D2
    summary: D2's rationale pins snapshot object counts already logged in the support log, and restates the repack-under-another-pair point that Open risks carries
---

### [MEDIUM] The low-speed abort is not classified as a transport failure

**Location:** `_lyx/discussion/decision-record.md` D3.

**Issue:** D3 sets `GIT_HTTP_LOW_SPEED_LIMIT`/`GIT_HTTP_LOW_SPEED_TIME` so that a dead HTTPS connection fails in about a minute, which is the #458 case.
When that limit fires, git itself exits with curl error 28 (`RPC failed; curl 28 Operation too slow. Less than 1000 bytes/sec transferred the last 60 seconds`, or `unable to access ...: Operation too slow`).
`IsTransportFailure` (`internal/gitexec/transport.go`) matches only `transportPhrases`: `could not resolve host`, `connection refused`, `connection reset`, `connection timed out`, `operation timed out`, `network is unreachable`, `failed to connect to`.
`operation too slow` is not among them.
D3 states that only "a timed-out command", meaning lyx's own wall-clock deadline whose message lyx authors, is reported true by `IsTransportFailure`.
The low-speed abort, the path #458 hits, therefore reaches a shed boundary as a hard, non-transient error and gets no transient re-step.
The best-effort status push still logs and continues, but any shed-boundary push or fetch stops as a repair-path failure instead.

**Fix:** In D3, state that the low-speed abort is a transport failure too, by adding curl's low-speed text (`operation too slow`) to the phrases `IsTransportFailure` matches, and add an acceptance criterion for it (see F5).

### [MEDIUM] gc.auto=0 does not cover git's auto-maintenance

**Location:** `_lyx/discussion/decision-record.md` D2.

**Issue:** Since git 2.29, the post-command trigger is `git maintenance run --auto` (`maintenance.auto`, default true), not `gc --auto` directly.
The local git is 2.53.0; its `git-maintenance` docs say `maintenance.strategy` "impacts manual maintenance, auto-maintenance as well as scheduled maintenance", and a `geometric` or `incremental` strategy runs `geometric-repack` or `incremental-repack`, each with its own `.auto` threshold that `gc.auto` does not govern.
`gc.auto=0` stops repacks only while the strategy is the default `gc` one.
An operator with a global `maintenance.strategy`, or a store that `git maintenance start` registered, still gets repacks under live steps.
The test fixtures already know this: `internal/gitkit/gitkit.go` and `internal/hubforge/hub.go` set both `maintenance.auto=false` and `gc.auto=0`.
D2's Goal is that no background repack lands under a live step, so it should switch off the trigger itself, not one of its tasks.

**Fix:** In D2, set `maintenance.auto=false` beside `gc.auto=0` on both stores, at the same two points.
Name `maintenance.*` in D6's PATTERN entry and in the acceptance criterion for D2's config.
State in D2's bound that scheduled maintenance an operator registers (`git maintenance start`) stays outside lyx's control.

### [LOW] D1's lock discipline is left open

**Location:** `_lyx/discussion/decision-record.md` D1.

**Issue:** Focus item 1 is settled as follows.
Today no go-git read calls another wrapped `Repo` method while it holds the lock.
`HeadContains` resolves its target through `commitByHash`, not `SHAExists`, and composite methods such as `Merge` call `CurrentSHA` and `MergeHeadPresent` one after another, never inside a read.
So "retries at most once per call" holds per helper invocation, and a composite caller gets one retry per read it makes, which is fine.
D1 still never states the rule that keeps this true.
`r.goGitMu` is a `sync.RWMutex`, which is not reentrant, and `goGit()` itself takes the exclusive lock (`internal/gitrepo/gogit.go:28`).
A helper that "holds the handle lock" and is `goGit()`'s only caller deadlocks if it locks before calling `goGit()`.
It also deadlocks if a wrapped closure ever calls another `Repo` read.
D1 does not say whether the helper holds the lock shared or exclusive either.
Today's reads hold `RLock` during walks and diffs (`blobread.go`, `merge.go`), and `Reindex()` needs the exclusive lock.
An exclusive lock around every whole read serializes all go-git reads on a handle, `PathRevisions` walks included.

**Fix:** Add the lock discipline to D1: `goGit()` is resolved before the lock, a wrapped closure never calls another `Repo` method, and the lock mode is chosen.
For example: a shared lock for the read, upgraded to exclusive only around `Reindex()` and the fingerprint update.

### [LOW] D3 does not bound what a timeout then triggers

**Location:** `_lyx/discussion/decision-record.md` D3.

**Issue:** D3 hands a timed-out command to "each caller's existing transport-failure path", and its bound lists only the commands that now fail.
The existing paths are bounded.
`shedtransient.Class` marks the failure `git-transport`, `shedengine` persists `StateFailed` with that class, and the shed driver stencil (`contracts/stencils/shed/shed-template-driver.md`, Transient re-step) gives it one immediate re-step, then hands back.
So a legitimate slow push or fetch costs at most two deadlines, about 20 minutes, and then a hand-back, never an unbounded loop.
The Decision does not record this, so the plan writer cannot check that the timeout stays bounded end to end.

**Fix:** Extend D3's bound: a timeout at a shed boundary gets the driver's one transient re-step and then hands back, so the worst case is two deadlines, and a best-effort push logs and continues.

### [LOW] Acceptance criteria miss three D2/D3 behaviors

**Location:** `_lyx/discussion/decision-record.md` `## Acceptance criteria`.

**Issue:** No criterion covers these three behaviors:
- D2 and Q3: an existing hub, whose stores lack `gc.auto=0`, gets it at its next pair removal, before housekeeping runs.
  The only config criterion is "after a pair is created".
- D3: `pull` joins `IsTransportFailure`'s subcommands.
  Today `remoteSubcommands` holds only `push`, `fetch`, `ls-remote` and `clone`.
- D3: a remote command carries the low-speed environment, and its abort classifies as a transport failure (F1).
  The criteria check only that a local command does not get it.

**Fix:** Add a criterion for each: the config is applied at removal on a store that lacks it; a `pull` transport failure classifies true; a remote command's low-speed abort classifies true.

### [LOW] The new gc PATTERN entry needs an explicit carve-out

**Location:** `_lyx/discussion/decision-record.md` D6 (and D2).

**Issue:** `PATTERN-fabric-destruction-chokepoint` makes `destroy.go` "the only file in `package fabricengine` permitted a destructive primitive".
D2's housekeeping runs `gc --auto` from another fabricengine file, and its prune deletes unreachable objects.
D2 argues this is outside the gate's model, which is sound, since nothing the gate checks is touched.
But D6's new PATTERN entry states only that housekeeping owns gc.
A later reviewer who reads the chokepoint entry would see a deleting primitive outside `destroy.go` with nothing in PATTERN that admits it.

**Fix:** Have D6's new entry state the exemption: object-store housekeeping prunes only what git's own expiry admits, and is not a destructive primitive under `PATTERN-fabric-destruction-chokepoint`.

### [LOW] The #447 tension is described differently in the two files

**Location:** `_lyx/discussion/support-log.md` Q6; `_lyx/discussion/decision-record.md` `## Auto-mode assumptions` and `## Open risks`.

**Issue:** Q6 says the #447 note's reuse expectation "is recorded as an open tension in the decision record's assumptions".
The record's assumption states a settled resolution instead: the board Goal governs D4 over the note's proposal.
The note's `Expected` is the operator's own statement, dated 2026-10-08, that a done slug is free and runs fresh.
After this task lands, that workflow still refuses, and the operator still has to rename the slug until `lyx-cleanup` lands.
The record says that nowhere a landing report would pick it up.
The plan writer needs nothing more for D4, which is decided, so this is a consistency gap, not a design one.

**Fix:** Align Q6's wording with the record ("resolved by an auto-mode assumption").
Add one Open risks line: a done slug stays unusable until `lyx-cleanup` removes finished statuses, so the operator's #447 expectation is not met by this task.

### [NIT] D2's rationale carries snapshot figures and a repeated point

**Location:** `_lyx/discussion/decision-record.md` D2 Rationale.

**Issue:** "about 8,600 unreferenced loose objects, above git's 6,700 threshold" is a point-in-time measurement.
The support log's Interview step 4 already records it, and the plan writer needs only the fact that the stores sit above the threshold.
The completeness-before-leanness test keeps that requirement; the counts carry none.
D2's "A gc there can still repack under another pair's live step; D1 makes that harmless" is restated by Open risks' second bullet, which adds the one new fact: an escape from D1's helper reopens #468.

**Fix:** In D2, say "above git's default auto-gc threshold" without the counts, and drop the repack-under-another-pair sentence from D2's rationale, which leaves it in Open risks.

## Focus verdicts not raised as findings

- Focus 2 (D2's bound across read paths): settled, not a defect.
  go-git is imported only under `internal/gitrepo`, so D1's helper covers every lyx go-git read.
  Agents' and the operator's git CLI rescan packs on a miss themselves.
  The prune is bounded by git's default expiry, which D2 keeps.
  The gap that remains is the trigger D2 disables, raised as F2.
