---
verdict: BLOCKING
findings:
  - id: F1
    severity: "BLOCKING"
    class: "design"
    location: "internal/burlerengine/cluster.go:451"
    summary: "Attached --config-env redirects can send an exempted scratch push to the target."
    fix: "In parseGitInvocation, parse the name and attached value of --config-env before the generic option branch and mark remote., url. and branch. overrides as redirecting. Add a row to TestFirstMutatingGitCommandScopesToForkRepositories with a safe recorded origin overridden by --config-env=remote.origin.pushurl=DEST; require a violation."
    origin: "lens:code-correctness"
  - id: F2
    severity: "BLOCKING"
    class: "design"
    location: "internal/burlerengine/cluster.go:324"
    summary: "Remote tracking can mark a destination safe while an external push URL remains active."
    fix: "Replace the single remote-safety boolean with separate fetch and push destination state in forkRepositories. A fetch set-url must preserve explicit push URLs; permit a push only when every effective push URL is proven safe. Treat conditional or unconfirmed URL updates conservatively. Extend the existing repository-scope test with external pushurl followed by safe fetch set-url, and a failing old-URL match."
    origin: "lens:code-correctness"
  - id: F3
    severity: "BLOCKING"
    class: "design"
    location: "internal/burlerengine/cluster.go:287"
    summary: "The new clone and push exemptions infer repository ownership from unverified lexical paths."
    fix: "In recordClone and leadsInto, stop treating an attempted clone or a descendant path as proof of a fork-created repository. Carry successful creation and canonical repository identity from the audit substrate, or keep ambiguous clone and push cases classified as mutations. Match the resolved destination repository itself, excluding symlinks into the target and failed clone destinations. Add these cases to the existing repository-scope test."
    origin: "lens:code-correctness"
  - id: F4
    severity: "MEDIUM"
    class: "design"
    location: "internal/burlerengine/cluster.go:736"
    summary: "A push from a scratch repository falsely establishes that the target repository was untouched."
    fix: "Keep targetUntouched false for an unresolved or external push destination regardless of its source. In fixViolationWayForward distinguish an unchanged local worktree from unknown destination refs and request destination inspection. Correct the external-destination row of TestAuditClusterRound so it no longer expects an unconditional no-target-change statement."
    origin: "lens:code-correctness"
  - id: F5
    severity: "MEDIUM"
    class: "design"
    location: "internal/shedadapters/burlerfix.go:295"
    summary: "Reportless failed attempts lose their rerun explanation when the producer is called again."
    fix: "Persist a rerun reason after a started fixer fails even when no report exists, keeping Report empty. Update archiveReport and the attempt-loop persistence condition accordingly, while distinguishing sessions that never started. Extend the producer rerun table with two reportless failed attempts followed by a new producer call, and assert the nonempty reason forwarded to that fixer."
    origin: "lens:code-correctness"
  - id: F6
    severity: "MEDIUM"
    class: "consistency"
    location: "contracts/stencils/burler/burler-fix-self-check.md:45"
    summary: "Fork instructions still prohibit the scratch Git probes that the changed guard permits."
    fix: "Update the self-check and reviewer-fork boilerplates, including the assistant fan, to permit Git mutations only in verified fork-created scratch repositories and pushes confined to those repositories. Narrow the inherited never-push instruction in burler-step-fix.md to preserve the target and its remotes while allowing that explicit probe exception."
    origin: "lens:code-affected-set"
  - id: F7
    severity: "MEDIUM"
    class: "design"
    location: "internal/shedadapters/burlerfix.go:172"
    summary: "Retiring an accepted review leaves its rerun record available to a later identical review."
    fix: "Clear FixerRerun whenever the accepted review is retired or replaced, in both fix-producer and review-producer round archive paths. Preserve the rest of the round-start snapshot. Extend the producer lifecycle tests with a recorded rerun, retirement, and a fresh accepted review with identical bytes; the new fixer must receive no retired rerun metadata."
    origin: "lens:code-affected-set"
  - id: F8
    severity: "MEDIUM"
    class: "design"
    location: "internal/shedadapters/burlerfix_test.go:628"
    summary: "Cancellation tests never assert the cancellation-specific rerun reason."
    fix: "Extend the existing cancellation case to make another producer call under a fresh context after the failed attempt. Assert that the next Fix receives the cancellation-specific reason and the actual archived report path, rather than the underlying runner error."
    origin: "lens:code-tests"
  - id: F9
    severity: "MEDIUM"
    class: "design"
    location: "internal/shedadapters/burlerfix_test.go:554"
    summary: "Retry tests observe the local rerun value but not persistence of the pre-attempt archive."
    fix: "Extend the existing rerun table with an attempt that writes a report and dies, then a retry that exits without writing another report. Construct a new producer over the same run directory and assert that its fixer receives the preserved archive path and an applicable rerun reason. Assert persistence independently of the immediate second Fix call."
    origin: "lens:code-tests"
  - id: F10
    severity: "MEDIUM"
    class: "consistency"
    location: "internal/shedadapters/roundsnapshot.go:33"
    summary: "The two-way merge declaration omits a tracked record-format change read by later binaries."
    fix: "Amend the landing summary's Merge danger declaration to account for the new persisted fixer_rerun field and its consumers, using the door classification required for a record-format change. Name the review and fix producers and snapshot consumers in the blast-radius description."
    origin: "lens:code-affected-set"
---

The committed range is `43fbea87e2d5c57e5b9fee1fd97de36fceddf54b..424f40c5d9ca9ec581129c4a15938e761aacd534`.
Both requested changes have implementations, but the guard exemptions do not reliably preserve the target boundary.

### [BLOCKING] Attached configuration redirects bypass the guard

**Location:** `internal/burlerengine/cluster.go:451`

**Issue:** The option lookup uses the complete argument, so `--config-env=remote.origin.pushurl=DEST` is skipped.
After a fork records a safe scratch origin, a push with this override retains that origin's safe boolean.
Git instead uses the destination supplied by `DEST`, which can be the target repository or its remote.
A read-only probe with `--config-env=url./target.insteadOf=REVIEW_PUSH_PREFIX` resolved `/scratch` to `/target`.
This spelling is valid Git syntax, unlike attached `-c`, which the installed Git rejected.

### [BLOCKING] A fetch URL update erases an unsafe push destination

**Location:** `internal/burlerengine/cluster.go:324`

**Issue:** `recordRemote` writes the same boolean for ordinary `set-url` and `set-url --push`.
Create scratch work and bare repositories, add the bare repository as origin, then set origin's push URL to the target.
An ordinary `set-url origin /scratch/bare.git` marks origin safe again, although its external push URL remains.
The subsequent push passes `pushMutates` and can change the target's refs.
A conditional `set-url` whose old-URL expression matches nothing also changes the audit's boolean despite Git rejecting it.

### [BLOCKING] Lexical ownership admits target mutations

**Location:** `internal/burlerengine/cluster.go:287`

**Issue:** `recordClone` records its destination without knowing whether Git created a repository there.
A failed clone into an existing nonempty `/srv/target` therefore exempts a later independent commit in that target.
The new push allowance also accepts every descendant of a recorded root through `rootContaining`.
A destination `/scratch/work/target.git` that is a symlink to `/srv/target.git` passes that lexical check.
Neither case establishes that the operation stays in a repository the fork created.
These are consequences of the new clone and push exemptions, rather than findings against unchanged parsing alone.

### [MEDIUM] The halt overstates what the audit knows

**Location:** `internal/burlerengine/cluster.go:736`

**Issue:** A push from a fork-created repository to `/srv/target.git` produces a violation with nonempty `pushFrom`.
That alone leaves `targetUntouched` true and renders a statement that no fork changed the target repository.
A successful push may have changed that repository's refs before this post-run audit executes.
The new test explicitly expects the incorrect statement for an external destination.

### [MEDIUM] Reportless retries become first runs after restart

**Location:** `internal/shedadapters/burlerfix.go:295`

**Issue:** Attempt 1 can die without a report, and attempt 2 then receives an in-memory explanation.
The loop persists that explanation only when an archive exists.
If attempt 2 also dies without a report, `archiveReport` returns before recording the failure.
A later call finds neither a report nor a record and forwards a zero `FixRerun` despite the earlier sessions.

### [MEDIUM] The permitted probe remains forbidden by its prompt

**Location:** `contracts/stencils/burler/burler-fix-self-check.md:45`

**Issue:** The self-check boilerplate requires read-only Git commands and prohibits any repository mutation.
`burler-cluster-fan.md:22` repeats that restriction, and `burler-step-fix.md` adds an inherited never-push rule.
The changed guard and package documentation now permit scratch pushes, but an obedient fork still cannot perform them.
This is an affected instruction surface of the requested permission change.

### [MEDIUM] A retired review's rerun metadata survives

**Location:** `internal/shedadapters/burlerfix.go:172`

**Issue:** `archiveWholeRound` archives the review, fixer report and acceptance record but preserves `FixerRerun`.
The review producer's archive set also omits the snapshot, and `recordRoundStart` keeps an existing snapshot.
After a halted fix records metadata and a later gate failure retires that review, a new review can have identical bytes.
Its digest then matches the retained record, so its fixer receives the old halt reason and archived report.
A content digest alone does not distinguish those accepted-review lifetimes.

### [MEDIUM] Cancellation wording is not checked across calls

**Location:** `internal/shedadapters/burlerfix_test.go:628`

**Issue:** The cancellation test asserts the returned context error and archive count only.
The new rerun table has no cancellation case.
Removing the cancellation reason assignment at `burlerfix.go:145` therefore leaves these checks passing.
This is the diff's own persisted behavior and accounts for the corresponding mutation survivor.

### [MEDIUM] Pre-attempt archive persistence has no assertion

**Location:** `internal/shedadapters/burlerfix_test.go:554`

**Issue:** The died-attempt row makes one `Call` and checks the second `Fix`'s in-memory `rerun` value.
That value is unchanged if the write at `burlerfix.go:295-296` is removed.
The halted-fix row checks a different write through `archiveReport`.
No row observes the pre-attempt record through a later producer instance, matching the reported surviving mutants.

### [MEDIUM] Merge danger omits the snapshot format change

**Location:** `internal/shedadapters/roundsnapshot.go:33`

**Issue:** The diff adds `fixer_rerun` and its nested fields to tracked round snapshot JSON.
Subsequent binaries read that record through `readRoundSnapshot`.
The landing summary nevertheless declares a two-way door with only burler as its blast radius.
The rubric explicitly disallows a two-way declaration for a record-format change another binary reads.
Tolerant JSON handling does not waive that review rule.

## Validation and focus coverage

The four lens reports are saved beside this report.
Read-only Git probes confirmed the attached configuration-override spelling; no target files were changed.
No test suite was rerun; the supplied reach, redundancy and mutation reports were used alongside source review.

The archive wrapper preserves its previous naming, collision and missing-file behavior.
The rendered work statement includes the reason and optional archive path and omits the block for a zero rerun.
Different review digests are excluded correctly.
No new timing waits or unrelated scope changes were found, and both bug-fix commits name relevant root causes.

## Rejected

- An attached `-c` bypass was discarded because the installed Git rejects that spelling.
- A clone of the target into a genuinely new scratch repository is not itself a target mutation.

