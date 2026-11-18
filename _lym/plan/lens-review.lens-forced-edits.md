```yaml
lens: forced-edits
outcome: reported
```

Forced-edits review of the 18-card plan: what each change forces outside the members its card names.

**Findings**

1. **BLOCKING — Card 3: `constructor` changes type, but two members that use it are not listed.** The card gives the unexported `constructor` type a worktree parameter and lists only `internal/shuttleengines#TestNew` among the tests that build constructor maps.
   - `internal/shuttleengines#TestKeepWarm` builds a `map[string]constructor{...}` literal (`internal/shuttleengines/keepwarm_test.go:71`) and passes it to `keepWarmWith` (`keepwarm_test.go:76`).
   - `internal/shuttleengines#keepWarmWith` calls those constructors in production code, so it is a caller too. Both stop compiling at card 3.
   - Fix: list both under card 3's `**Edit:**`.

2. **BLOCKING — Cards 14 and 15: the completion-signal tripwire is not listed.** `completionSignalScannedFiles` covers `wait.go`, `attach.go`, `stop.go` and `apierrorretry.go` (`internal/shuttleengine/completionsignal_enforcement_test.go:34`). The tripwire fails on any new negative-verdict return site in those files.
   - Card 14 puts `stopForRelaunch` in `stop.go`, and it reports the stop's outcome. It also adds died and timeout ends to the relaunch.
   - Card 15's `settleStaleRelaunch` finalizes `OutcomeDied` from `Wait`, and it adds a case to `dispositionCandidate` in `attach.go`.
   - Only card 12 lists `internal/shuttleengine#TestCompletionSignal_NegativeVerdictReturnSites` and `#TestCompletionSignal_FileContractCallSites`. Cards 14 and 15 add new sites after it, so their own card gates fail.
   - Fix: list both tests on cards 14 and 15, with the pinned sites they add.

3. **MEDIUM — Cards 7 and 8: the old single-directory resolvers are left behind.** Card 7 moves the live readers to `sessionProjectDir`; card 8 moves `CheckResume` to the resolved directory and `checkResumable` to a list of registry directories.
   - After that, `internal/shuttleengine/claudeengine#claudeProjectDirFor`, `#projectsRoot` and `#sessionRegistryDir` (`audit.go:105`, `transcriptread.go:26`, `resume.go:55`) have no production caller.
   - `claudeProjectDirFor` keeps only `TestClaudeProjectDirFor_EncodesNonAlnumBytes`, which makes it test-only code. `PATTERN-test-only-code` is enforced by a test.
   - The file doc comments of `resume.go` and `transcriptread.go` still say the readers read the registry and projects "under the home directory".
   - No card deletes the functions or retargets the test and comments. Fix: delete them on card 8, or on card 7 for the ones card 7 frees, and retarget the test to `projectDirUnder`.

4. **MEDIUM — Card 7 (D8 "installed-plugins lookup takes the launch's effective directory"): the manifest still reads the old directory.** Card 7 changes `installedPlugins` and `sessionScopeFor`. But the launch manifest builds its user setup through `internal/shuttleengine/claudeengine#Claude.manifestInputsFor` / `#Claude.resolvedCandidates` (`manifest.go:169`, `manifest.go:316`). That setup gets `claudeDir: c.claudeDir` (`manifest.go:324`), which is empty, so `UserSetupSource.resolveConfigDir` (`usersetup.go:112`) falls back to the lym process's own `CLAUDE_CONFIG_DIR` or `~/.claude`.
   - The process's own value is the calling pane's account, so a session exported to account B gets a manifest listing account A's plugins and instruction files.
   - Fix: list these members on card 7 (or card 6) and pass them the launch's effective directory.

5. **LOW — Card 5: no production site for the `battenCLI.pickAccount` field.** The card says the field is "set in production", but `battenCLI` is built at `internal/battencli/cli.go:438` (`Command`), `arm.go:519` and `cancelcmd.go:413`, and none of those members is listed.
   - Fix: either say in the Intent that `battenCLI.wire` sets the field, or list `internal/battencli#Command`.

6. **LOW — Card 3: `TestEngineWorktree` has no package stated.** `internal/hubgeom` holds both `package hubgeom` and `package hubgeom_test` test files (7 and 3). The handle `plan:internal/hubgeom#TestEngineWorktree` is tier1 on `hubforge`, but the Intent does not say which test package it goes in. An external-package test would need the unit `internal/hubgeom_test`.

Tool calls made:
- grep for the callers of every re-signed member: they were not in the pre-read.
- awk for the enclosing function of each `shuttleengines.New` and `RecordReaders` call line: the pre-read gave only line numbers.
- `lym quarry resolve` over the cited glyphs (all resolve), plus a grep for `constructor` uses in the `shuttleengines` tests and a read of `primeShuttle`: the glyph spellings and the test uses of `constructor` were not in my context.
- grep for test package clauses, `battenCLI` construction sites, `orchengine.Session` implementers, run-record key fixtures and the tripwire's scanned files: none of these were in my context.
