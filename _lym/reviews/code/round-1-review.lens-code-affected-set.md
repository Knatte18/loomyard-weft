```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set, listing per change what the diff forces outside its own lines.

**Per change: the set found, covered or not**

1. `post-checkout.sh` `PARENT`/`BASE` expansion.
   - The only embedder is `postCheckoutScript` in `hook.go`, via `go:embed`. It is also concatenated into the chain wrapper, so the wrapper inherits the change.
   - `SCRIPT_DIR` appears nowhere else in Go, docs or fixtures. The `sandbox/posix/*.sh` matches in the grep are unrelated suites.
   - No Go or markdown file asserts the old `dirname`/`basename` lines. The other `dirname` hits are the launcher scripts, which the diff does not touch.
   - The `case "$0"` expansion and the root-path edge cases are the code-correctness lens's.
   - Covered.
2. `chainedHookPreamble` expansion.
   - It is a constant used only in `chainUserHook`. Hubs already carrying the sentinel are left alone by the installer, so old wrappers keep working.
   - The doc line added at `hook.go:39` is self-contained.
   - Covered.
3. `newRecordsFixture` to `CopyHub`.
   - Every `TestRecordsReplay` row reaches `CopyHub`, because all of them call `newRecordsFixture`. That includes the `EndedSet` rows at `replay_integration_test.go:467,484,502,520,538`.
   - `pushclasses_integration_test.go:40` already builds its own `recordsFixture` with `CopyHub`, so the pattern matches.
   - Covered.
4. adoptcli `NewHub` to `CopyHub`.
   - The grep shows no `NewHub(` left in `internal/adoptcli`. The five sites are converted: `adopt_integration_test.go:58,216,239`, `check_integration_test.go:30` and `deploy_integration_test.go:78`.
   - Covered.
5. Remaining `NewHub` sites, checked for reach beyond D5's list.
   - The diff touches none of the other `NewHub` sites: `worktreeseam:197`, `boltpullfirst:41`, `verbs_regenerate:95`, `weftgit_exclude_test:199`, `housekeeping:172,197`.
   - No `internal/hubreconcile` file is in the diff.
   - Covered.

**Findings**

- [LOW] [scope] `internal/fabricengine/boltpullfirst_integration_test.go:41`, `verbs_regenerate_integration_test.go:95`, `housekeeping_integration_test.go:172,197`
  - The D5 acceptance criterion says each kept `NewHub` site "names the clone or wiring behavior that needs it". I did not confirm that a reason is recorded at these sites, and the diff adds none. They are outside D5's explicit list (`newRecordsFixture`, adoptcli, hubreconcile), so this may not count as a gap.
  - Evidence: my second call returned no `//` comment lines in the 6 lines before those `NewHub` calls in these three files. `worktreeseam:197` does have a reason, in the comment at lines 183-185.
  - Severity is LOW because D5 names only the three site groups.

- No other finding. `summary.md` is accurate against the diff. Both commit messages match their diffs, and no `Pattern-Departure:` trailer is needed. The diff declares no PATTERN changes.

**Tool calls**
1. grep for `dirname`/`basename` in `internal/fabricengine` and for `chainedHookPreamble`, `post-checkout.sh` and `SCRIPT_DIR` repo-wide: my context did not show who else mirrors or asserts the old text.
2. Read the top of `weftgit_exclude_test.go` and the comment lines before the `NewHub` calls in three files: my context did not show whether the kept sites carry a reason.
