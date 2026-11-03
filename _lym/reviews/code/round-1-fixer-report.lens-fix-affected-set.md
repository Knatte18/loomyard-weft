```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fixes for F1–F4 cover every member of the sets their premises name.

No findings.

- F1 (probe sites): `AnchoredPathHasContent` has one production caller, `sharedbootstrap.go:170`. It is guarded by `!syncOptions.SkipGit`, so every caller is covered.
- F2 (docs describing the first-seed clear): the grep shows `doc.go:54-55` and `overview.md:600` updated, and `sharedbootstrap.go` has the `seedAndCommitBootstrap` and `ensureFrictionDirAfterSeed` docs updated. The remaining first-seed mentions are accurate as they stand:
  - `sharedbootstrap.go:138` and `:163` describe the clear itself.
  - `:233` says a first seed's clear is committed.
  - `:266` is a log message.
- F4 (renamed row): no `FrictionDirectoryPresent` string remains anywhere in the repo.
- F3 (coverage): the new probe test has all four rows the review named, and the re-entry subtest asserts the resume note is tracked.

Tool calls made:
1. Grep for first-seed and clear-friction phrasing across go and md files, to check for stale doc sites.
2. Grep for `AnchoredPathHasContent` and `FrictionDirectoryPresent`, to check callers and leftover names.
