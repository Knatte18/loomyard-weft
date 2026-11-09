```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test that pins it, or none.

**Behaviour → pinning test**

- Content keys keep the four parts (file, `Type.Method` or `(package)`, hash, ordinal): `TestKeyBlocks` (`blockkey_test.go:38`). It checks `A.Do` against `B.Do` with a pointer receiver, ordinals 0 and 1 for identical blocks, and the package-level name.
- A shifted block keeps its key: `TestKeyBlocks`, the `shifted` loop.
- An edited block gets a new key: `TestKeyBlocks`, the `edited` check.
- `KeyBlocks` errors on a line outside the file: `TestKeyBlocks`, the final check. These have no test:
  - a column outside the file;
  - an end before the start;
  - an unparseable file;
  - generic receivers (`receiverTypeName`'s `IndexExpr` and `IndexListExpr` cases);
  - blocks in two files at once (the per-file ordinal grouping).
- `Judge` (containment, greedy removable set, covering tests, and the skipped, failed, out-of-process, unresolved and no-blocks reasons): `TestJudge` (`verdict_test.go:23`), four table rows, covering every reason.
- `ParseProfile`, `PackageBlocks` (own files only, nested package excluded) and `CoveredPositions`: `TestParseProfile` (`profile_test.go:12`), including four malformed lines.
- `BuildBinary` arguments and `-tags`: `TestBuildBinaryAndListTests` (`runner_test.go:38`). The `CGO_ENABLED=1` extra environment variable is not asserted, and a failing build carrying stderr has no test.
- `ListTests` filtering out the `ok` line, and omitting `-tags` when the tags are empty: `TestBuildBinaryAndListTests`.
- `RunAlone`: `TestRunAlone` (`runner_test.go:67`). Its four rows are pass with profile, fail without profile, skip without profile, and pass without profile as an error. It also asserts the `-flag=value` argument form. These have no test:
  - a skip that did write a profile, so the profile is parsed (`runner.go:108-119`);
  - a malformed profile in `RunAlone`;
  - a binary that does not start;
  - removal of a stale previous profile.
- `Map` round trip, not found, unknown key read without refusal and kept on rewrite: `TestWriteMapReadMap` (`coveragemap_test.go:14`). These have no test:
  - `BuildFailure` set and read back (the written map never sets it);
  - a second write replacing the old tests and blocks whole;
  - `MapPath` placing a nested package path under the directory.
- `lymdirs.TestCoverDirName`: no test. It is a constant, and the map verb that composes the path is outside this diff.
- `judgePackage` splits into package and module views (D2), and a candidate unique only in another package is not module-contained: `TestJudgePackage` (`redundancy_test.go:12`). It asserts the verdict struct and the `| yes | no |` row.
- `judgePackage` with a module-contained candidate (`moduleContained: true` from `moduleVerdicts[i]`): **none**. The `yes` in the column is only hand-built in `TestRenderPackage`.
- `judgePackage` for a kept candidate (listed under Kept, classification unchanged): **none** through `judgePackage`. `TestJudgePackage` keeps `TestWide`, which is not a candidate. `TestRenderPackage`'s kept row uses hand-built verdicts.
- `renderPackage` for the module-wide column, Kept, No coverage and the error section: `TestRenderPackage`, four rows.
- `renderReport`'s header prose: static prose, exempt.
- `measurePackage`, which does the list, build, spawn scan, run-alone loop, and sets a missing scan entry to `Unresolved`: **none**. It runs real `go` and is not injectable. The pre-change `redundancy_test.go` had no test of it either: the diff removes only `TestClassify`, `TestRunAloneArgs` and `TestParseProfile`, which moved to `TestJudge`, `TestRunAlone` and `TestParseProfile`.
- The `-pkg` splice and flags: unchanged code with its existing tests, and the diff touches none of them.
- `ScanSpawns` and `ScanKeeps`: moved with their tests (`spawnscan_test.go`, `keep_test.go`), re-pointed. I did not re-read them.
- The `cmd/lym` allowlist re-point: guarded by `tierpurity_test.go` itself.

**Findings**

- **LOW, design**, `internal/testcover/runner.go:108-119` and `runner_test.go:77-87`: no row for a skipped test that wrote a profile. This is documented behaviour ("or skipped after writing one") whose profile parse is unpinned. Fix: add a stand-in row that skips and writes a profile.
- **LOW, design**, `internal/testcover/coveragemap.go:62` and `coveragemap_test.go:20`: `TestWriteMapReadMap` neither round-trips `BuildFailure` nor shows a second `WriteMap` replacing the first whole. Both are D4 fields or behaviours. Fix: set `BuildFailure` in a second written map with fewer tests, and assert the read-back equals it.
- **LOW, design**, `cmd/testtiming/redundancy.go:86-96` and `redundancy_test.go:29`: no `judgePackage` row where a candidate is also contained module-wide, and none where a kept test is a candidate. The `moduleContained` copy and the keep copy are pinned only through hand-built verdicts. Fix: add a `runs` entry covered in both views (`yes`) and put a keep on a candidate in `TestJudgePackage`.
- **LOW, design**, `internal/testcover/blockkey.go:57-76,164-167` and `blockkey_test.go`: no test of blocks from two files, a generic receiver, or a column outside the file. Fix: add rows or assertions to `TestKeyBlocks`.

I raise `measurePackage`'s missing test as no finding: the pre-change tree had none, and the no-test-spawns-`go` constraint explains it.

**Tool calls**
1. `git diff ... -- cmd/testtiming/redundancy_test.go`, filtered to function lines: my context did not show which tests the old file held.
