```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens, running each changed function or branch against a concrete input. I found no defect.

**List (function or branch, case run, verdict)**

- `runStateExemptRels`, returns `["_lym/shed", "_lym/loom/friction", "_lym/loom/friction-*"]`. `LoomDurableDirRel` is `_lym/loom` (`internal/loomengine/config.go:291`). Correct.
- `LoomFrictionArchivesRelPattern`, name `_lym/loom/friction-20261103-120000/loom-halt.md`. The glob is `_lym/loom/friction-*`. Git's default pathspec is fnmatch without `FNM_PATHNAME`, so `*` crosses `/`. It matches the archive directory and everything beneath it. The doc comment says the same. Correct.
- `LoomFrictionArchivesRelPattern`, the live friction directory `_lym/loom/friction`. The glob needs a `-` after `friction`, so it does not match the live directory. The separate literal entry covers it. Correct.
- `LoomFrictionArchivesRelPattern`, non-archive sibling `_lym/loom/friction-foo.md` (a file) or `_lym/loom/frictionX`. The first is over-matched, the second is not. This agrees with `IsLoomFrictionArchive` (`config.go:369`), which accepts any name with `friction-` plus at least one character. Directory and file siblings alike are exempt. Unreachable in practice, because lym writes only directories there. Not a defect.
- `exemptPathspec`, root anchor `"."`. `filepath.Join(".", "_lym/loom/friction-*")` gives the same string. The pathspec is `[".", ":(exclude)_lym/shed", ":(exclude)_lym/loom/friction", ":(exclude)_lym/loom/friction-*"]`. Correct.
- `exemptPathspec`, subpath anchor `"backend"`. `ScopedPathspec` (`internal/fabricengine/fabric.go:224`) joins to `backend/_lym/loom/friction-*`. Correct. `filepath.ToSlash` runs after the join, so Windows separators are normalised. The glob has no backslash. Correct.
- `exemptPathspec`, empty exempt list. It returns nil and the probe runs unscoped. Unchanged and correct.
- `Clean`, weft status with a deleted tracked file (`loom-halt.md` committed, then removed). `git status` applies pathspec limits to index and worktree entries alike, so `:(exclude)_lym/loom/friction` hides ` D _lym/loom/friction/loom-halt.md`. The same holds for the file moved into an archive, untracked, under the glob. Correct. The test pins this: unscoped `Clean` must be dirty first (`preflight_integration_test.go:458`), then the scoped check passes.
- `Clean`, git literal-pathspec mode silently turning the glob into a literal. `gitexec.Run` sets no `GIT_LITERAL_PATHSPECS`. Only specific callers use `RunWithEnv` with it (`integrateweft.go:116`, `destroy.go:2181`, `anchormove.go:112`, `anchortarget.go:119`), and none sits on the `Clean` path. The glob is live. Correct.
- `Clean`, git spawn failure. `dirtyReason` wraps it and `Clean` returns `(false, "", err)`, as before. Correct.
- Settle-log half of the entry (`_lym/shed/<slug>/go-settle.jsonl`). `RunsRootRel` is `_lym/shed` (`internal/shedrun/paths.go:358`), and `SettleRecords` sits under `RunDir` beneath `RunsRoot`. Already exempt before the diff, and the test's `runState` list still covers it. Correct.
- The doc-comment-only change in `warpclean.go`. It matches the code, which passes entries through unescaped. Correct.
- `TestCheckResolved_LymOwnStateIsExempt`, root and `backend` anchors. The test commits a halt note through `h.PrimeRecords()`, deletes it, writes an archive file and a new note, writes the run state, then asserts three things in order:
  1. Unscoped `Clean` is dirty.
  2. `CheckResolved` reports no failure.
  3. A file under `_lym/plan` fails `CheckWorktreeClean`.

  The flow is correct. Whether `PrimeRecords` is the same worktree that `Clean` probes as the weft is a fixture fact I did not re-verify. If it were another checkout, assertion 1 would fail, so it cannot pass silently.

**Findings:** none.

**Tool calls**
1. `grep` for `LITERAL_PATHSPECS` and the like, plus `LoomDurableDirRel`: my context lacked whether `gitexec.Run` forces literal pathspecs, and the value of `LoomDurableDirRel`.
