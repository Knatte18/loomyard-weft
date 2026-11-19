---
round: 1
focus:
  - "internal/gatecli/tierrun.go runTier: the answer key (parent entry) asks only for a gate check failing a tier run that leaves new untracked, non-ignored files; settle whether each part of the diff, including the split into runTier and runTierCommands and the copies.go change, serves it and whether anything else is changed."
  - "internal/gatecli/tierrun.go runTier and strayfiles.go checkStrayFiles: settle whether files the gate itself writes in the worktree (a prebuilt lym binary, copies' scratch, run records, locks, .scratch or _lym output) can show up as stray and fail a healthy run, and whether tiers that run in parallel in one worktree can blame each other's files."
  - "internal/gatecli/tierrun.go runTier: settle what happens to the check on a cancelled or timed-out run, a run whose result carries a refusal, and a run with no invocation, and whether the exit code and outcome set in checkStrayFiles stay consistent with what the callers and the summary line read from tierResult."
  - "internal/gatecli/strayfiles.go untrackedWorktreeFiles: settle whether `git ls-files --others --exclude-standard -z` run at request.root sees the right tree (nested repositories, submodules, root below the repository top, paths with odd bytes) and whether a git failure after a successful first listing is handled soundly."
  - "internal/gatecli/strayfiles_integration_test.go and the mutation report: settle whether TestRunTierStrayFiles asserts independent literals rather than values computed as the code computes them, and whether the surviving mutants at strayfiles.go:21 and tierrun.go:74, 75 and 83 name behavior a test could assert (the not-listed-before-the-run path, the no-invocation skip, the refusal skip)."
  - "internal/gatecli/doc.go `# Stray files`: settle whether each claim (exit code kept on a failing run, the line to stderr and the log before the summary line, `--copies` listed once, unlisted worktree not checked) is borne out by the code."
  - "The commit messages of the range: settle whether any claim of agreement or improvement lacks figures, and whether a Pattern-Departure or Loosened-Check trailer is present and valid."
---

Round 1 focus for the stray-file gate check.
The hardenshed test fix is out of scope by the parent's recorded decision, so a missing hardenshed change is no finding.
