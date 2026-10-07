---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D4 (hard cut of WEFT_SKIP_GIT and WEFT_SKIP_PUSH)
    summary: D4 removes the old skip env vars with no fallback but does not state what that lets through, namely that a process still setting only the old name now runs git and pushes it used to skip
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D2 Bound, last two sentences
    summary: The sentence about a step 12 rejection deleting the warp branch follows the SkipGit/SkipPush sentence and reads as its consequence, but under those options step 12's push never runs and the origin branch is kept
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md Goal, first bullet
    summary: The Goal says a rollback leaves no warp branch on origin, which D2's own bound contradicts for an adopted, pre-existing or unprobed branch
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/discussion/support-log.md Interview turn 5
    summary: The phrase 38-file identifier set pins a tally that no file in the repo states and that goes stale silently
---

### [LOW] D4's hard cut of the skip env vars states no bound

**Location:** `_lyx/discussion/decision-record.md`, D4 ("The renames are hard cuts: no alias, no fallback read of the old env var, key or trace detail").
**Issue:** `WEFT_SKIP_GIT` and `WEFT_SKIP_PUSH` are guards: set, they stop `EnvSyncOptions` and `SpawnDetachedPush` from running git or pushing.
After the cut, a process that still exports only the old name silently runs git and pushes for real.
The rubric's attack-surface question requires the Decision that removes or downgrades a guard to state what it can now let through and what bounds it; D4 states only "no fallback read", and the Auto-mode assumption that nothing outside the repository consumes them is not a bound.
I read the repository: the only setters of the old names are test files, so the bound exists, but the Decision does not say so.
**Fix:** Add a Bound line to D4: the cut lets through only git and push work in a process that still sets the old name; the repository's setters are tests, renamed in the same commit, and the skip variables are a test and sandbox switch with no production consumer.

### [LOW] D2's Bound ends with a sentence its antecedent does not support

**Location:** `_lyx/discussion/decision-record.md`, D2 Bound, the sentences beginning "Under `SkipGit` or `SkipPush`" and "A real non-fast-forward or hook rejection at step 12".
**Issue:** The second sentence opens with "therefore" directly after the `SkipGit`/`SkipPush` sentence.
Read in order, it says that under those options a step 12 rejection deletes the warp branch from origin.
That is false in both halves: in `add.go` step 11's push is unconditional, but `pushWeftBranch` returns before pushing under `SkipGit` or `SkipPush`, so step 12 cannot reject there, and the rollback there deletes the local branch only.
The sentence is true only for the probed, non-skip case, and it is the statement of what the weakened rule now lets through, so a plan writer must read it unambiguously.
**Fix:** Put the step 12 sentence before the skip sentence, drop "therefore", and state it as the non-skip consequence of the origin deletion; keep the skip sentence last as the case where only the local branch goes.

### [NIT] Goal over-claims what a rollback removes on origin

**Location:** `_lyx/discussion/decision-record.md`, Goal, first bullet ("a rollback leaves no warp branch behind, locally or on origin").
**Issue:** D2 deletes the origin branch only when the probe ran and found it absent and this `Add` attempted step 11's push.
An adopted branch, a pre-existing branch pushed forward, and any rollback under `SkipGit` or `SkipPush` keep origin's branch by design.
The Goal states the unconditional form; the acceptance criterion narrows it to the default empty `branch_prefix`, and the Goal does not.
**Fix:** Reword the bullet to "a rollback leaves no warp branch this `Add` created, locally or on origin".

### [NIT] Support log pins an unverifiable tally

**Location:** `_lyx/discussion/support-log.md`, Interview turn 5 ("has grown past the 38-file identifier set").
**Issue:** No file in the repository states a 38-file set; the number records a past state and cannot be re-derived.
**Fix:** Drop the count: "has grown past the identifier set into env vars, ...".

## Focus verdicts

- **D3, how the Commit route learns it runs in the prime.**
  Not a defect.
  `Fabric` holds only the two paths and two `gitrepo.Repo` handles, no `Location`, and D3 does not claim otherwise.
  `Fabric.Commit` resolves its own `*lyxcwd.Location` with `lyxcwd.ResolveWorktree(f.warpPath)` before classifying paths, which is the `Location` D3 says the route decides from.
  `PushPairAnchored` takes `l` as a parameter, and the `sync` verb holds `l` in its closure, so D3's premise for those two matches the code.
  `SpawnDetachedPush` omits an empty `--warp-path` flag, and the bypass child pushes the records side alone through `CoalescePushBothAt`, which supports an empty code-side path.
- **D3, a prime-name resolution failure on the Commit route.**
  Not a defect.
  D3 places the failure before the commit: `Fabric.Commit` returns the error before `commitBothSides` takes the lock, so the fire-and-forget spawn after a landed commit never runs on an unresolved prime, and the guard cannot fail open.
  The acceptance criteria cover the route twice: `lyx fabric commit` in the prime pushes the records side only, and a resolution failure makes `Fabric.Commit` fail with no push spawned.
  Every in-repository `Fabric.Commit` caller (the three fabric verbs and webster's `fabricSync`) passes routing-set paths, which classify to the records side, so D3's rationale that no code-side commit is the call's own holds for them.
- **D2, the origin-deletion condition under `SkipGit` or `SkipPush`.**
  Not a defect.
  Step 11's push is unconditional in `add.go`, while both probes run only when neither option is set, so an unprobed pre-existing branch is a real case.
  D2 handles it: the rollback receives the probe's tri-state outcome, only "absent" qualifies, and "not run" deletes the local branch alone.
  `probeWarpLeftover` returns the same empty tip for "absent" and for "present at or behind `HEAD`", which is why D2 needs the carried outcome and does not derive it from the return value; the Decision says so.
  An acceptance criterion covers the skipped-probe rollback.
  F2 concerns only the wording of the sentence that follows.
- **D2 and its Constraints, recording a refused or failed remote deletion.**
  Not a defect.
  `deleteRemoteBranch` appends `KindRemoteBranchDeleted` only after `DeleteRemoteBranchLeased` succeeds, so a refused lease or a failure appends nothing, which matches `PATTERN-mutation-record` ("an executor appends only after it observably changed state").
  D2's "only a deletion that happened is recorded in `rec`" and the Constraints' "success-only entry" agree with the code and with each other.
