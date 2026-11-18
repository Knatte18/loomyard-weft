```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I traced what each change in `71d6ee1a..HEAD` forces outside the lines it touches.

**Per change, the set found and whether it is covered**

1. **Python row always installed on Claude and Codex** (`shellSwitchInstalled`, `switchInstalled`). This also covers Claude's new `announced` predicate and Codex's `denyNotice(…, in)` signature.
   - Callers: `codexengine.go:168-179` covered; the `wantCreelHead` and `denyNotice` call sites in `prepare_test.go` and `usersetup_test.go` covered.
   - Tests:
     - the manifest tests (`manifest_test.go` in both engines) are covered: the `switched` var at claude :109 and codex :75 is still used only for the every-switch row, where the python row is present, so it is still right;
     - `TestBuildSettings_DenyToggleMatrix`, `TestBuildSettings_NoAgentHookAndForkGuard` and the `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch` counts are covered;
     - `TestBuildDenyNotice_MatchesInstalledDenies`, `TestInstalledGuards_CoverEveryRow` and `TestGuardEntries_RenderEveryRowWithItsCodexText` are covered;
     - the golden `settings-golden.json` is regenerated: covered.
   - Docs:
     - claudeengine `doc.go` covered;
     - codexengine `doc.go` covered;
     - shellguard `doc.go` and `rows.go` covered;
     - `shuttleengine/config.go` covered;
     - `template.yaml` covered;
     - `docs/overview.md:683` (Claude paragraph) covered;
     - `docs/overview.md:717` (Codex paragraph) **not covered**, see F1;
     - `internal/shuttleengine/spec.go:79-80` still says "toggleable via … deny_python". That remains true of the deny's effect, so it is not a finding.
   - The refusal text `pythonRefusal` (`rows.go:81`) names `deny_python: false` as the way forward. It is still correct, and now takes effect live. Covered.
2. **New hidden verb `hook-python-allowed`** (`PythonAllowedHookVerb`, `hookpython.go`, the `cli.go` skip list and `AddCommand`).
   - PATTERN-cli-cobra:
     - the audience is `internal` and the command is `Hidden`: covered;
     - the lower-case `Short` has no final period: covered;
     - `RunE` checks `ShouldAbort` first: covered;
     - errors go through `output.Err`: covered;
     - the `Long` cap is fixed per commit `ac646ddc`: covered.
   - Standalone wiring goes only through `cliwire`, and `standalonestate.Derive` is called only in `cliwire` (`standalone.go:55-58`): covered.
   - PATTERN-cwd-resolution: `ResolveWithin` is called by a read-only verb a session runs: covered.
   - PATTERN-config-layers: the config loads through `shuttleengine.LoadConfig`, which goes through configengine: covered.
   - PATTERN-test-only-code: `StandaloneStateDir` has a production caller: covered.
   - Tests: `hookpython_test.go` and `hookpython_standalone_test.go` cover it.
3. **Hook commands that name the lym binary** (`pythonAllowedCheck` in both engines): see F2.
4. **configcli help line** (`configcli.go:484`). `internal/configcli/doc.go:13` says the help names the switch's effect on "both the hook and the system-prompt sentence". The new line says it "lets Python through and drops the notice at launch", so the doc still holds. Covered.
5. **The `shellguard.SwitchDenyPython` doc comment.** No other reader asserts the old text. Covered.
6. **Commit messages.** No `Pattern-Departure:` trailer appears in the range, and the directive declares no PATTERN changes. Nothing to check.

**Findings**

- **F1 — LOW, consistency, `docs/overview.md:717`.** The Codex paragraph still says Codex installs the shellguard rows "the python and publish rows following their switches". Under this diff the python row is installed in every Codex launch, and only its notice follows `deny_python`. The Claude paragraph at :683 was updated; the Codex one was left stale.
  - Evidence: `docs/overview.md:717`; `internal/shuttleengine/codexengine/guards.go` `switchInstalled`, where `SwitchDenyPython` returns `true`.
  - Fix: reword it to say the python row is installed in every launch and asks `lym shuttle hook-python-allowed` on a match, its notice following the value at launch.
- **F2 — MEDIUM, consistency, `internal/shuttleengine/claudeengine/settings.go:339` and `internal/shuttleengine/codexengine/guards.go:38`.** The new hook commands name a bare `lym` on PATH. PATTERN-child-processes says a command naming the lym binary for "a hook" resolves it through `selfexec.LymBinary`, and `internal/selfexec/doc.go:8` repeats that hooks use it.
  - No engine hook does this today. Claude's `creelMailCommand` (`settings.go:107`) and `creelWaitCommand` (:183) use bare `lym`, and so do Codex's `hook-edit` (`editrecord.go:28`), `hook-batching` (`batchingnudge.go:94`) and the creel commands (`command.go:205-206`). `grep selfexec.` over `internal/shuttleengine` and `internal/shuttleengines` finds no hit.
  - So the new hook follows the engines' established convention, and the gap predates this diff.
  - The handler should decide whether the rule's literal text binds here, which would make this BLOCKING by the constraint rule, or whether the uniform precedent makes it a pre-existing pattern gap, which is not this diff's to fix.

Tool calls:
1. grep of `.md` docs and the manifest tests' `switched` var: to find the docs and test variables describing the python guard, which the manifest did not cover.
2. grep of non-Go files, `EditHookVerb` registries and the tests naming the new verb: to find the stencils, specs and fixtures reading the changed outputs.
3. Read of `spec.go:74-84`, the start of the golden diff and the `shuttleengine` doc mentions: to check the remaining `deny_python` statements for staleness.
4. grep for `selfexec.` in the engines and for hooks in the `selfexec` doc: to check the PATTERN-child-processes rule against the engines' practice.
