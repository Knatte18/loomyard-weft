---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D1 (Lock discipline)
    summary: Splitting the failing read (shared lock) from the fingerprint check (exclusive lock) opens a window in which a concurrent caller's reindex makes the fingerprint look unchanged, so the read is never rerun and a present object is reported absent
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D2 (housekeeping paragraph) and D6 (D2 entry carve-out)
    summary: The premise that housekeeping gc removes only unreachable objects is false; git gc also expires reflogs and prunes stale worktree admin entries, which the destruction-chokepoint carve-out does not name
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D6 (D2 entry)
    summary: D6's PATTERN entry says only fabricengine's housekeeping runs gc on hub stores, dropping D2's lyx-code qualifier, so it contradicts D2's bound that manual git gc and scheduled git maintenance still run
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D3 (opening paragraph and Rationale)
    summary: The premise that every production git call goes through runCore is false; tools/deploy/main.go runs push and fetch through its own exec.Command, so those stay unbounded and the record neither covers nor scopes them out
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D3 (Bound, last sentence)
    summary: The worst case of two deadlines holds per failing command, not per step; a step whose best-effort remote calls log and continue can wait out one deadline per remote call before the transient one, times two
---

### [MEDIUM] D1's lock split lets a concurrent reindex suppress the retry

**Location:** `_lyx/discussion/decision-record.md`, D1, "Lock discipline".

**Issue:** D1 runs the read under the shared lock.
On object-not-found, it releases that lock and takes the exclusive lock for the fingerprint check, `Reindex()` and the fingerprint update.
Its first paragraph reruns the read only when the fingerprint changed ("checks the pack fingerprint, reindexes when it changed, and reruns the whole read once").
`sync.RWMutex` has no upgrade, so a gap separates the failed read from the check.
Suppose two callers on one `Repo` both fail against the stale index, and caller A reindexes and updates `lastPackFingerprint` first.
Caller B then sees the current fingerprint equal to `lastPackFingerprint`, takes the "unchanged" branch and returns object-not-found without rerunning.
`SHAExists` or `HeadContains` then answer "absent" for an object that is present.
That is the #468 class of error this task is meant to close.
Today's `lookupObjectRetrying` (`internal/gitrepo/gogit.go:51-74`) holds the exclusive lock across the lookup and the check, so the window does not exist there.
The `Repo` type doc (`internal/gitrepo/gitrepo.go:40-42`) restricts only concurrent writes, so concurrent reads on one `Repo` are in contract.
The record does settle the order against `goGit()` and rules out reentrant calls, so the round's deadlock question is answered.
The race is a separate gap.

**Fix:** State the retry condition in a form the split cannot defeat.
For example, the helper records `lastPackFingerprint` before the read and reruns whenever the fingerprint under the exclusive lock differs from that snapshot, reindexing only if it also differs from the current `lastPackFingerprint`.
Alternatively, D1 keeps today's exclusive lock across read and check, and says why.
Add an acceptance criterion for two concurrent readers over a stale index.

### [LOW] Housekeeping gc deletes more than unreachable objects

**Location:** `_lyx/discussion/decision-record.md`, D2's housekeeping paragraph ("the prune it runs removes only unreachable objects past git's own expiry"), and D6's D2 entry carve-out.

**Issue:** `git gc`, `--auto` included when it fires, also runs `git reflog expire` and `git worktree prune`, subject to `gc.reflogExpire*` and `gc.worktreePruneExpire`.
On the hub's stores, `worktree prune` deletes stale `worktrees/<name>` admin directories, and those stores back fabric's pairs.
The carve-out from `PATTERN-fabric-destruction-chokepoint` rests on the claim that housekeeping removes only unreachable objects.
That claim is incomplete, so the PATTERN entry would carry a false premise.

**Fix:** Name the full deletion set in D2 and in the D6 carve-out: unreachable objects, expired reflog entries and stale worktree admin entries, each past git's default expiry.
Alternatively, pin `gc.worktreePruneExpire=never` on the housekeeping command line if worktree metadata must stay under the destruction gate, and state that choice.

### [LOW] D6's PATTERN entry overstates D2's ownership claim

**Location:** `_lyx/discussion/decision-record.md`, D6, second bullet.

**Issue:** D2 says "the only lyx code that runs `gc`".
Its bound says a manual `git gc` still works and scheduled maintenance from `git maintenance start` stays outside lyx's control.
D6's entry text drops the lyx qualifier: "only fabricengine's housekeeping at pair removal runs `gc` or writes `gc.*` or `maintenance.*` on them".
Read as a PATTERN line, it claims nothing else ever gcs the stores.
That contradicts D2 and leaves the out-of-control sources unnamed in the one place reviewers check.

**Fix:** Word the D6 entry as "the only lyx code that…".
Have it name what stays outside lyx's control: the operator's manual `git gc` and registered scheduled maintenance.

### [LOW] Not every git remote call goes through `runCore`

**Location:** `_lyx/discussion/decision-record.md`, D3's opening paragraph ("which every production git call already goes through") and its Rationale ("all production git runs through `runCore`").

**Issue:** `tools/deploy/main.go:358-363` (`gitOut`) runs `git push` (line 107) and `git fetch` (line 149) through its own `exec.Command`, outside `gitexec`.
`update-plugins.sh` runs that code on the operator's production route.
CLAUDE.md holds `tools/` to PATTERN like everything else.
Those calls keep hanging on a dead connection after D3 lands, and nothing in the record says so.
`hubforge` and `gitkit` also spawn git directly, but they are test kits.

**Fix:** Either bring `tools/deploy`'s remote calls into D3's scope, or list them under Out with the reason, such as "operator-attended".
Then make the premise accurate: "every git call in `internal/`".

### [LOW] D3's end-to-end bound is per command, not per step

**Location:** `_lyx/discussion/decision-record.md`, D3, Bound's last sentence ("so the worst case is two deadlines").

**Issue:** The driver's transient re-step (`contracts/stencils/shed/shed-template-driver.md:96-97`) and `shedtransient.Class` (`internal/shedtransient/shedtransient.go:27`) bound how often a failing step reruns.
They do not bound how many remote commands one step attempt runs.
Under D3, a best-effort push that times out logs and continues, so a later remote call in the same attempt can wait out its own deadline.
On SSH, where the low-speed limit does not apply, that is 10 minutes per call.
One attempt can therefore cost several deadlines before the transient error returns, and the re-step doubles that.
"Two deadlines" understates the worst case.

**Fix:** State the bound as one deadline per remote command, times the step's remote calls, times two attempts.
Alternatively, accept that wording explicitly and add the residual to Open risks.

## Focus directive verdicts

- D3, low-speed abort classification: settled.
  D3 states that `operation too slow`, curl's error-28 text, joins `IsTransportFailure`'s phrases, and an acceptance criterion tests it.
  Today `internal/gitexec/transport.go:20-28` lacks the phrase and `remoteSubcommands` lacks `pull`, and the record adds both.
  Not a defect.
- D2, `maintenance.auto`: settled for the auto trigger.
  D2 sets `maintenance.auto=false` and explains why `gc.auto` alone misses strategy-driven tasks.
  D2's bound names scheduled maintenance as outside lyx's control.
  D6's entry does not, which is F3.
- D3, end-to-end bound: stated, and it cites the correct driver rule, but it is per command (F5).
- D1, lock mode and order against `goGit()`: stated, with `goGit()` called before the lock, the shared read and the exclusive reindex, and no nested `Repo` calls, so no deadlock.
  The split it introduces opens the race in F1.
