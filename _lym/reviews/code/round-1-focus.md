---
round: 1
focus:
  - "Target-to-test correspondence: trace where invocation.targets comes from in internal/gatecli and every command shape a prebuilt_lym tier can hold ({targets}, {test_targets}, literal packages, no placeholder, args after --, --card, per-module invocations). Does invocationsUseLymBinary see every package whose tests the command runs?"
  - "Routes to the binary: independently enumerate every way a test can depend on the exported prebuilt variable or the binary, including non-Go consumers (tier commands, scripts, hub config), child go test or go run calls, nested modules and testkit helpers. Compare the result with the change description's claims and its file and directory counts."
  - "Path handling in isDirectoryTarget, resolveRunDir and impactset.TestsUseLymBinary: check relative versus absolute root, '.' and '..' inside the kit's own directory, file or flag-like targets, trailing slashes, symlinks, platform separators, and whether the lymbinDir suffix match behaves as the doc comments say. Does each uncertain case build?"
  - "Both call sites (runInvocations and runTierCopies): check that skipping the build keeps timeout, retry, copies and cleanup behavior intact, and what a skipped run does with a LYM_PREBUILT_LYM already present in the incoming env."
  - "Test coverage of the new behavior: does the --copies path or any positive build case against an existing directory get exercised? Do the existing prebuild subtests still check what they claim when they build only because their target directory is absent?"
  - "Docs and claims: check the gatecli and impactset package docs, pattern/PATTERN-test-speed.md, other comments or docs that still say a prebuilding tier always builds, and the change description's Merge danger door and blast radius against the diff."
---

Seed for round 1, over `a30022546ba518bffb7bed30a5f792ff05d19e66..HEAD`.
The answer key is the decision record's board entry alone.
It asks that every route to the prebuilt binary be checked and counted, so that no test loses it,
and that a call over the whole set still build it.
