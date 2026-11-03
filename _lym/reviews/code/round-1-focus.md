---
round: 1
focus:
  - "internal/loomcli/sharedbootstrap.go step 2b/3: the friction clear runs only on a genuine first seed, and the commit probe runs on every invocation. Settle whether a resume (ErrSeedExists) with untracked or tracked friction notes now commits those notes into the fabric, and whether that contradicts the 'resume notes are worth reading' intent or the goal's 'clear inherited halt' expectation."
  - "internal/fabricengine/commitweftpaths.go AnchoredPathHasContent: settle whether entryMatchesWeft with weftCommitPathspec matches a tracked-but-deleted directory path and an untracked non-ignored file, and whether its error and ignored-file cases keep git from refusing the pathspec in step 3."
  - "Goal coverage: settle whether the diff handles both the case where the clear removes tracked notes and the case where the friction directory is untracked-only or empty, and whether Preflight's worktree-clean check passes afterwards on a fabric with a committed halt note."
  - "Tests: settle whether TestBootstrapCommitPaths and the TestSmokeStepBootstrapWiring additions cover the new behavior without duplicating each other, and whether AnchoredPathHasContent and the withFriction probe have any coverage."
  - "docs/overview.md, internal/loomcli/doc.go and the commit messages: settle whether each claim matches the diff, in particular that the clear is committed 'with its seed' for both `start` and `step`."
---

Seed focus for a diff touching the first-seed friction clear and its commit.
