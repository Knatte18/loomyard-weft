---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: >-
      The review offered two changes: render the launch's config dir into each engine's check command,
      or fall back to the standalone state dir when no hub resolves.
      I made the second, inside the verb alone, so no hook command or engine signature changes.
      The verb prefers the hub's layers when the worktree has a Bolt checkout, otherwise reads the state dir
      derived for the worktree root or the cwd when it exists, and otherwise keeps its old result or error.
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
---

# Round 1 fixer-report

## What changed

- F1, F6: `internal/shuttlecli/hookpython.go` writes its failure through `output.Err` and carries `clihelp.SkipStencilSeedAnnotation`.
  The `Long` text says the failure is a JSON error object.
  The `OutsideAWorktree` row now parses the output as the JSON error object.
- F2: `shuttleConfigDirs` in the same file picks the config directories.
  A hub worktree with a Bolt checkout gets the hub's layers.
  Otherwise the state directory `standalonestate.Derive` gives for the worktree root, or the cwd when no worktree resolves, is used when it exists.
  A new test, `TestHookPythonAllowedVerb_FollowsAStandaloneRunsStateDirConfig`, writes `deny_python` in that state dir's worktree-layer file and expects exit 0 for false and exit 1 for true.
- F9: a row `AllowedFromBelowTheAnchor` runs the verb from a new directory under the prime worktree.
- F3: `internal/orchcli/denypython_integration_test.go` wires the orch CLI over a hub with `deny_python` at the default layer, prepares a launch on the wired claude engine with the wired shuttle config,
  and asserts the notice holds every python-switch sentence only for true while the settings carry the python hook's live check for both values.
- F4, F5: `testtiming:keep` markers on `TestBuildSettings_DenyToggleMatrix` (which gained a one-line doc comment) and `TestInstalledGuards_CoverEveryRow`.
- F7: the `SwitchDenyPython` doc in `internal/shellguard/rows.go`.
- F8: the `lym config` help line in `internal/configcli/configcli.go`.
- F10: the panic message in `internal/shuttleengine/codexengine/guards.go`.
- Self-check follow-ups: the python-row text in both engines' `doc.go`, the `PythonAllowedHookVerb` doc in `internal/shuttleengine/engine.go`
  and both `pythonAllowedCheck` comments now name the shuttle config the launch read, with the standalone state dir, instead of the hub only.
  The stale "switches it off" sentence in `claudeengine/doc.go` is reworded.
  The `lym config` help sentence ends with "Set it from the prime:" so that clause attaches to the command.
  The F3 test collects every python-switch notice rather than the last one.

Gate repair: the gate refused eight comment sites the round added, for wrapped breaks or lines of 120 characters or more.
Each is now one sentence per line, split at a clause boundary or rewritten as shorter sentences:
the header and doc comment of `internal/orchcli/denypython_integration_test.go`, the `SwitchDenyPython` doc in `internal/shellguard/rows.go`,
the standalone test doc in `internal/shuttlecli/hookpython_test.go`, the python-row paragraphs of both engines' `doc.go`,
and the `pythonAllowedCheck` comments in `claudeengine/settings.go` and `codexengine/guards.go`.
Only comment text changed.
A second gate run still refused the last two lines of the `pythonAllowedCheck` comments in `claudeengine/settings.go` and `codexengine/guards.go`,
which broke a sentence over two lines at a comma.
Each now says the verb finds the config from the session's worktree, and in a second sentence that it reads the standalone state directory when no hub holds the worktree.

A later verify run failed four checks, repaired as follows:
- `TestDeriveCallerSet_CliwireOnly`: `standalonestate.Derive` is pinned to `internal/cliwire`, so `hookpython.go` no longer calls it.
  `cliwire.StandaloneStateDir` is a new derive-only function, and the verb calls it for `cliwire.RepositoryRootOf(cwd)`, the same repository-root lift a standalone launch applies.
  This replaces the F2 fallback's target of the worktree root or the cwd; the rest of the F2 rule is unchanged.
- `TestTierPurityFailures` and `TestTierTags_OneTierPerTestFile`: the tier-0 standalone test moved out of the tier-1 `hookpython_test.go` into `hookpython_standalone_test.go`.
- `TestCLIForm_EveryCommand`: the `lym config` help passed its 2000-byte cap, so the `deny_python` line is now one shorter line:
  "deny_python=false lets Python through and drops the notice at launch, from the prime:".

Gate before that repair: `lym gate test` tier 0 and tier 1 over `./internal/shuttlecli`, `./internal/orchcli`, `./internal/configcli`, `./internal/shellguard`,
`./internal/shuttleengine`, `./internal/shuttleengine/codexengine` and `./internal/shuttleengine/claudeengine` passed with no failure.

## Self-check

- fix-code-correctness, LOW: the F3 test kept only the last python-switch notice. Fixed as proposed.
- fix-code-correctness, LOW: the `lym config` help clause "from the prime" attached to the notice drop. Fixed as proposed, as "Set it from the prime:".
- fix-consistency, LOW: `claudeengine/doc.go` still said `deny_python` switches the row off. Fixed as proposed.
- fix-consistency, LOW: the verb's doc, both engine docs and both check comments said hub only. Fixed as proposed, naming the config the launch read.
- fix-consistency, LOW: `hook-batching` and `hook-edit` keep plain-text stderr while the new verb emits JSON. Disputed, see below.
- fix-proposal, LOW: help line ambiguity. Merged with the fix-code-correctness help finding; fixed.
- fix-proposal, LOW: the F3 test lets the claude engine read the machine's Claude Code directory. Deferred, see below.
- fix-affected-set, LOW: the `claudeengine/doc.go` twin. Merged with the fix-consistency finding; fixed.
- fix-affected-set, LOW: the hub-only wording in the docs. Merged with the fix-consistency finding; fixed.
  `docs/overview.md` and `shuttlecli/cli.go` name no hub as the config source, so they stay as they are.

## Disputed

- Ground `every fix worse`, the fix-consistency lens's note on the sibling verbs.
  The review called the plain-text stderr of `hook-batching` and `hook-edit` existing debt that licenses no new site, and the new verb now follows the rule.
  Converting the siblings changes two verbs outside this change set; their hooks discard stderr too.
  Evidence: `internal/shuttlecli/cli.go:81` lists the three hook verbs together as exempt from the pre-run, and the finding itself proposes no change.

## Deferred

- fix-proposal LOW, the F3 test reads the machine's Claude Code directory through the real claude engine.
  Removing that read needs a seam on the engine set or the claude engine, which is production code outside this round's write surface and outside the finding's files.
  The test's assertions name only the python sentence and the python hook's verb, so the machine's contents cannot change its result.

## Rulings

None.
