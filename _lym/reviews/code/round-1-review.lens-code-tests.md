```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if the behaviour broke.

**Per behaviour, the test that pins it:**

1. **Shipped patterns match the base name at any depth, and near-misses pass.** Pinned by `TestMatcher_Match` rows `EnvAtRoot` through `OrdinaryFileAllowed`.
2. **A pattern with a slash matches the whole path; any other pattern matches the base name.** Pinned by `TestMatcher_Match` rows `AddedSlashPatternMatchesWholePath` and `AddedSlashPatternSkipsDeeperPath`.
3. **`Match` returns shipped patterns before added ones.** Not pinned. `ShippedPatternWinsOverAdded` adds `.env`, the same string as the shipped pattern, so swapping the order returns the same `.env`.
4. **`New` refuses a malformed pattern and names it.** Pinned by `TestNew_RefusesMalformedPattern`.
5. **The zero `Matcher` uses the shipped patterns alone; `Hits` keeps input order; `Describe` renders hits.** Pinned by `TestHits_DescribeNamesEachPathAndPattern`.
6. **`Config.SecretPathMatcher` splits on whitespace, merges with the shipped list, and wraps `ErrInvalid` naming the key.** Pinned by `TestLoadSecretPathMatcher` rows.
7. **`secret_paths` is closed to the local layer (`CategoryGuard`).** Pinned by the `TestLoadConfig` local-key row `secret_paths`.
8. **The template carries an empty `secret_paths`.** Pinned by `TestConfigTemplate`.
9. **`Fabric.Commit` refuses a new secret file on warp and on weft, names path, pattern and way forward, and lands nothing.** Pinned by `TestCommit_SecretPath_RefusedAndCommitsNothing` rows `Warp` and `Weft`.
10. **`Fabric.Commit` refuses a pattern the hub's `fabric.yaml` `secret_paths` adds, and a malformed one surfaces as an error.** None.
11. **A refused `Fabric.Commit` stages nothing and leaves both indexes untouched.** None. The test checks only HEAD SHAs and `CommitResult`.
12. **`Fabric.Commit` skips the weft check when the weft side is not taken (`SkipGit`).** None.
13. **`NewPathsIn` lists untracked and staged-added paths; never tracked edits or ignored files; scoped to its pathspec; sorted.** Pinned by `gitrepo_test.go`, step "NewPathsIn lists the untracked…".
14. **`NewPathsIn(nil)` or an empty pathspec returns nothing, not the whole repo.** None.
15. **`AddedFilesBetween` lists inserts only, never modifications or deletions.** Pinned by the `gitrepo_test.go` step "AddedFilesBetween lists only…".
16. **`AddedFilesBetween` counts a rename as an add under the new name (documented).** None.
17. **`record-batch` fails the batch on a shipped or hub pattern, names path and pattern, names the reset step and the recover step, and goes terminal failed; a clean batch records.** Pinned by `TestRecordBatch_SecretFileFailsBatch` rows `ShippedPattern`, `HubPattern` and `NoSecretFile`.
18. **`record-batch` fails "whatever its report says", that is for FAILED or NEEDS_PARENT reports too, before the evidence re-run and post-batch checks.** None. Only an OK report with no policy findings is exercised.
19. **`failOnSecretPaths` diffs from `bs.StartSHA`.** None. `fakeGit.AddedPaths` ignores the base and keys on head only.
20. **`webstercli` wires the hub matcher from `fabric.yaml` into `RecordDeps.SecretPaths`, maps a load failure through `configLoadError`, and uses the shipped patterns alone in standalone mode.** None. No test outside `websterengine`, `fabricengine` and `gitrepo` mentions secret paths.
21. **`realGit.AddedPaths` delegates to `addedPaths`, which wraps `AddedFilesBetween`.** Covered for the primitive by row 15; the wrapper is a pass-through.
22. **CLI help text and doc prose.** Exempt as static prose.

**Findings:**

- **MEDIUM | design | `internal/webstercli/wiring.go:230`, `internal/webstercli/recordbatch.go:114-120`:** The hub-pattern plumbing into `record-batch` is untested. If `wireHub` stopped setting `c.secretPaths`, or the verb dropped `SecretPaths: secretPaths` from `RecordDeps`, hub patterns would be silently ignored with no failing test. The `fabric.yaml` load-failure refusal is also unreached by any test, although PATTERN-refusal-way-forward requires a test that reaches each new webster refusal. Evidence: the grep finds no `*_test.go` under `internal/webstercli` mentioning `secret_paths`, `secretPaths` or `SecretPath`.
- **MEDIUM | design | `internal/fabricengine/commit.go:141-147`:** No test drives `Fabric.Commit` with a hub `secret_paths` pattern or a malformed one. Replacing `cfg.SecretPathMatcher()` with a zero `secretpath.Matcher{}` keeps every test green. Evidence: `commit_integration_test.go:133-134` uses shipped patterns only; `TestLoadSecretPathMatcher` calls `LoadSecretPathMatcher`, not `Commit`.
- **MEDIUM | design | `internal/fabricengine/commit_integration_test.go:155-172`:** The documented "stages nothing, leaves both indexes as it found them" guarantee (`commit.go:177`; summary "stages nothing and lands nothing") has no assertion. A refusal placed after `git add` would leave HEAD unchanged and pass. Evidence: the test checks only `result.*Committed` and `CurrentSHAForTest`.
- **LOW | design | `internal/gitrepo/gitrepo.go:422-424`:** The empty-pathspec early return in `NewPathsIn` is unpinned. Without it, `git ls-files --others --` with no paths lists the whole checkout. A weft-only commit would then be refused on an unrelated untracked warp secret. Evidence: the `gitrepo_test.go` step calls only `NewPathsIn([]string{"."})` and `NewPathsIn(fresh.txt, absent.txt)`; the `Weft` row of the fabric test plants no untracked secret in warp.
- **LOW | design | `internal/websterengine/recordbatch.go:518-522`:** The claim "fails whatever its report says, before the evidence re-run and post-batch checks" is not pinned for FAILED or NEEDS_PARENT reports. Moving the check below the NEEDS_PARENT return at line 544 would record a secret-carrying batch as waiting with no test failing. Evidence: `recordbatch_test.go:1107` writes only `validReport(fx.HeadSHA)`.
- **LOW | design | `internal/websterengine/recordbatch.go:629`:** The base argument `bs.StartSHA` is unpinned. `fakeGit.AddedPaths` ignores the base (`gitfake_test.go:346`), so passing the wrong base, for example the report head's parent, passes. A row keying `added` on the (base, head) pair would pin it.
- **LOW | design | `internal/secretpath/secretpath_test.go:732`:** `ShippedPatternWinsOverAdded` cannot detect a reversed order, because the added pattern is the same string `.env` as the shipped one. An added pattern that also matches but differs, such as `.e*`, would pin "shipped patterns first".
- **LOW | design | `internal/gitrepo/gitrepo.go:450-451`:** "A path renamed between the two counts as added under its new name" is documented with no row.
- **BLOCKING | consistency | `internal/websterengine/recordbatch_test.go:1458-1513`:** `TestRecordBatch_SecretFileFailsBatch` and its subtests never call `t.Parallel`, and no comment at the test or atop the file names process-global state. PATTERN-test-speed "Tests run in parallel" requires one or the other. The fixture is per-test: `newRecordFixture` uses `t.TempDir` and a fresh `newFakeGit`, and the helper file shows no `Setenv`, `Chdir` or `logger` call. Several older sibling tests in the file share the omission, but this test is new in the diff.

**Tool calls:**

1. Grepped `*_test.go` for `secret_paths`, `secretPaths` and `SecretPath`, and read `newRecordFixture` with a scan of its helper for process-global calls. My context did not show whether any `webstercli` test covers the new wiring, or whether the record fixture touches global state.
