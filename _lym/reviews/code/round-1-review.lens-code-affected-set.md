```yaml
lens: code-affected-set
outcome: reported
```

Task: list the set each change forces outside its own lines, and report findings, for the code-affected-set lens.

**Set per change**

- `bootstrapCommitPaths(location, withFriction)` has one signature change.
  - Its only caller is `sharedbootstrap.go:171`, which is updated. Its only test is `TestBootstrapCommitPaths`, rewritten at `sharedbootstrap_test.go:185-211`. The file header at `sharedbootstrap_test.go:12` still names it correctly. Covered.
- `seedAndCommitBootstrap` step 3 now stages the friction directory.
  - Its callers `start` and `step` both reach it. Its doc comment at `sharedbootstrap.go:58-61` still lists only "the seed, the status file, and the origin record". Not covered (F1).
  - `ensureFrictionDirAfterSeed`'s doc at `sharedbootstrap.go:240-246` says a resume leaves the directory untouched because its notes are worth reading. Step 3 now probes on every call, so resume-time notes get committed. Not covered (F2).
  - `docs/overview.md:600` and `loomcli/doc.go:54` describe only the first-seed clear commit, not the commit of resume notes. Partly covered (F2).
- `AnchoredPathHasContent` is a new export with one caller.
  - It is not a pair-aware verb and is fabric-vocabulary-neutral, consistent with `CommitAnchoredPaths`. No PATTERN entry conflicts. Covered.
- PATTERN.md and `pattern/`:
  - The diff changes neither, and no declared PATTERN changes were listed for me. `PATTERN-friction-capture` has no background file under `pattern/`. Nothing is owed.
- Commit messages: the four commits carry no `Pattern-Departure:` trailer. Nothing is owed.
- Existing tests asserting the old behaviour:
  - `friction_test.go:50` tests only `ensureFrictionDirAfterSeed`, which is unchanged.
  - `smoke_bootstrapwiring_test.go:136-144` ("re-entry keeps friction notes") still holds, because notes are kept on disk. It does not check whether they are committed.

**Findings**

- F1, LOW, consistency, `internal/loomcli/sharedbootstrap.go:58-61`: the `seedAndCommitBootstrap` doc comment says step 3 commits only the seed, status file and origin record. It now also commits the friction directory whenever it holds content. Evidence: lines 58-61 against 160-171.
- F2, LOW, consistency, `internal/loomcli/sharedbootstrap.go:240-246`, `docs/overview.md:600`, `internal/loomcli/doc.go:54`: the probe in step 3 runs on every invocation, so a resume's untracked or tracked friction notes are also committed into the fabric. The `ensureFrictionDirAfterSeed` doc says the resume branch leaves the directory untouched. The overview and `doc.go` describe the commit only as part of the first-seed clear. Fix: state in these places that step 3 commits whatever the friction directory holds on every invocation, or restrict the probe to a first seed. Evidence: `sharedbootstrap.go:160-171`, `sharedbootstrap.go:240-246`.
- F3, NIT, consistency, `internal/loomcli/sharedbootstrap_test.go:187`: the test comment says the friction directory is committed "only when it exists". The code's condition is "holds tracked or untracked content", which an existing empty directory does not meet. Evidence: `commitweftpaths.go:104-109`.

**Tool calls**
1. A grep over the repo for `bootstrapCommitPaths`, the first-seed clear wording and the friction docs, plus `ls pattern` and `git diff --name-only -- PATTERN.md pattern`. My context lacked the full caller, test and doc set for these symbols and the PATTERN diff status.
