---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomcli/sharedbootstrap.go:164
    summary: The friction probe runs git ls-files in the weft worktree even under FABRIC_SKIP_GIT, where the commit it guards is a no-op.
    fix: In seedAndCommitBootstrap step 3, skip the AnchoredPathHasContent probe when fabricengine.EnvSyncOptions().SkipGit is set (leave withFriction false), or pass the SyncOptions into AnchoredPathHasContent in internal/fabricengine/commitweftpaths.go and return false, nil when SkipGit is set.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/sharedbootstrap.go:58
    summary: Step 3 now commits friction notes on every invocation, resumes included, but the seedAndCommitBootstrap doc, the ensureFrictionDirAfterSeed doc, docs/overview.md and loomcli/doc.go describe only the first-seed clear.
    fix: In sharedbootstrap.go add the friction directory to the list of committed paths in the seedAndCommitBootstrap doc comment (lines 58-61), and state in the ensureFrictionDirAfterSeed doc (lines 240-246) that a resume leaves the notes on disk and step 3 still commits them. In docs/overview.md:600 and internal/loomcli/doc.go:54 say that every start or step commits whatever the friction directory holds, with a first seed's clear being the case that matters.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/fabricengine/commitweftpaths.go:107
    summary: AnchoredPathHasContent has no test of its own, and the false branch that keeps git from refusing a matchless pathspec, and the resume commit of untracked notes, are pinned by no test row.
    fix: Add a table-driven test in internal/fabricengine over a real records worktree with rows for a tracked file deleted on disk, an untracked file, an ignored-only directory and an empty directory, asserting AnchoredPathHasContent. In the re-entry subtest of TestSmokeStepBootstrapWiring (internal/loomcli/smoke_bootstrapwiring_test.go:136), assert whether the planted resume note is committed after the second step.
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/sharedbootstrap_test.go:187
    summary: The TestBootstrapCommitPaths comment says the friction directory is committed only when it exists, but the condition is that it holds tracked or untracked content.
    fix: Reword the comment to say the friction directory is committed only when it holds tracked or untracked content; rename the FrictionDirectoryPresent row to FrictionDirectoryHasContent.
---

### [LOW] Probe runs under FABRIC_SKIP_GIT

**Location:** internal/loomcli/sharedbootstrap.go:164
**Issue:** `CommitAnchoredPaths` returns early under `SkipGit`, but `AnchoredPathHasContent` calls `git ls-files` unconditionally.
In an environment with no usable weft git the bootstrap now fails at `bootstrapStageCommit` for a commit that would have been skipped.

### [LOW] Resume commits friction notes; docs describe only the first-seed clear

**Location:** internal/loomcli/sharedbootstrap.go:58
**Issue:** The probe runs on every invocation, so an `ErrSeedExists` resume with tracked or untracked notes commits them under the message `loom: seed session bootstrap for <slug>`.
The resume intent is kept on disk, so this does not contradict the resume-notes-are-worth-reading intent, and it matches the self-heal the step 3 comment states.
It is a behavior beyond the first-seed clear that the `seedAndCommitBootstrap` and `ensureFrictionDirAfterSeed` docs, `docs/overview.md` and `internal/loomcli/doc.go` do not say.

### [LOW] Missing coverage for the probe

**Location:** internal/fabricengine/commitweftpaths.go:107
**Issue:** The first-seed smoke subtest drives the probe only through the true branch (tracked note deleted).
The re-entry subtest also has a true probe.
The false branch, the ignored-only case and the resume commit have no test row.

### [NIT] Test comment misstates the condition

**Location:** internal/loomcli/sharedbootstrap_test.go:187
**Issue:** An existing but empty friction directory does not meet the probe's condition, so only exists is inexact.

## Rejected

- The goal-coverage question for a tracked-but-deleted friction directory is not a defect: `ls-files --cached` lists the deleted tracked note, so the probe is true and `git add -- <dir>` stages the deletion; verified in `entryMatchesWeft` and `StageAndCommit`.
- The untracked-only or empty friction directory case is not a defect: the first-seed clear removes untracked notes, the probe is false for the recreated empty directory, and git never receives a matchless pathspec.
- An ignored-only friction directory is not a defect: `--exclude-standard` makes the probe false, so the pathspec is omitted.
- Mid-goal: the goal's second expectation (a child's start clears an inherited halt and commits that removal before Preflight) is what the diff implements; no scope finding.
