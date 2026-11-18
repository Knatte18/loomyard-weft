```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I traced what reads, calls, asserts or describes it, and judged whether the diff leaves that set correct.

**What each change reaches, and whether it is covered**

1. **The python row is now installed in every launch, and its notice is decided separately** (claudeengine `shellSwitchInstalled`/`shellSwitchAnnounced`, codexengine `switchInstalled`/`switchAnnounced`).
   - Covered: `buildDenyNotice`, `denyNotice`, both `manifest.go` notice and guard candidates (which read the artifacts as written), `codexengine.go` Prepare, the init() panic guard, the manifest tests, `settings_test.go`, `guards_test.go`, `prepare_test.go`, `usersetup_test.go`, the settings golden, `docs/overview.md:683`, both engines' `doc.go`, `shellguard/doc.go:50`, the `config.go` field doc and the `template.yaml` comment.
   - Also checked and still correct: `shuttleengine/doc.go:75`, `spec.go:80` ("toggleable"), `claudeengine/doc.go:544`, `languageconfig/doc.go:36`, and the `pythonRefusal` steer in `shellguard/rows.go:79` (its way forward, "set deny_python: false", still holds).
   - Left wrong: `shellguard/rows.go:21` (`SwitchDenyPython` doc) and `configcli/configcli.go:484` (help text). See F1 and F2.
2. **New hidden verb `hook-python-allowed` and the `PythonAllowedHookVerb` constant.**
   - Covered: the `shuttlecli/cli.go` pre-run skip and `AddCommand`, the `cli.go` file header, `engine.go`, both engines' hook commands, `docs/overview.md`, `hookpython_test.go`, and both hook-run tests.
   - Nothing else registers or lists its sibling hidden hook verbs. A search for `hook-edit` and `EditHookVerb` found only the same set of places.
3. **Hook commands that call bare `lym`.**
   - The `selfexec.LymBinary` rule in `PATTERN-child-processes` guards against re-executing the test binary.
   - The existing hooks (`editrecord.go`, `batchingnudge.go` and the creel hooks) already call bare `lym` from a hook, so this is no finding.
4. **The `guardInputs` field doc and the `installedGuards` panic.**
   - The field doc and the function doc are updated.
   - The panic message is not. See F3.
5. **Repository rules.**
   - `PATTERN-shell-guard-table` (every engine renders every row) still holds.
   - `PATTERN-refusal-way-forward` lists `python` only as a denied form, which is unchanged.
   - `PATTERN-config-layers`: the meaning of `deny_python` (deny or allow) is unchanged, so no reinterpretation is needed. `deny_python` is `CategoryGuard`, so it is closed to the local layer, and the verb's `LoadConfig` respects that.
   - No PATTERN changes are declared, and no `Pattern-Departure:` trailers appear in the range's commits.
   - The commit messages match the diff.

**Findings**

- **F1 — LOW, consistency.** At `internal/shellguard/rows.go:21`, the `SwitchDenyPython` doc still reads "marks the row the python switch of the shuttle config turns off". The row is now installed in every run, and the switch only decides the hook's live answer and whether the notice appears at launch. The diff updated the same claim in `shellguard/doc.go:50` but left this one. Evidence: `internal/shellguard/rows.go:21`, and `claudeengine/settings.go` `shellSwitchInstalled` (`SwitchDenyPython` returns true).
- **F2 — LOW, consistency.** At `internal/configcli/configcli.go:484`, the user-facing `lym config` help says "Python guard off for a hub (hook and notice)". Setting `deny_python=false` no longer removes the hook. The installed hook now lets Python through on its next call, and the notice is dropped only for sessions launched afterwards. Evidence: `internal/configcli/configcli.go:484-485`, and `template.yaml`'s new `deny_python` comment.
- **F3 — NIT, consistency.** At `internal/shuttleengine/codexengine/guards.go`, `installedGuards` now panics when a switch is missing from either `switchInstalled` or `switchAnnounced`. Its message still says only "so the row would stay uninstalled", although the updated doc comment says "uninstalled or unannounced". Claude has a separate message for the announced case (`claudeengine/settings.go` init, "would stay unannounced"). Evidence: the diff hunk at `codexengine/guards.go` around lines 675-677.

One item belongs to another lens: in standalone launches (`standalonegeom.ReedGeometry`, where PaneCwd is the target and the config lives in the state dir), the new verb resolves a different config than the launch read. That is for code-correctness or the handler.

Tool calls:
1. Grepped `deny_python`, `DenyPython` and `SwitchDenyPython` across the repo to find readers and descriptions the manifest lacked.
2. Read `shellguard/rows.go:15-25`, `shuttleengine/doc.go:70-82`, `spec.go:76-84`, `claudeengine/doc.go:538-548` and `configcli.go:478-490` to judge the hits from call 1.
3. Read the `PATTERN-child-processes` LymBinary bullet and searched for python mentions in the rule files, to judge the hook's bare `lym`.
4. Read `PATTERN-refusal-way-forward` lines 15-30 and searched specs and stencils for deny_python, to check the rule and its wording against the change.
5. Grepped for the sibling hidden verbs to find any registry or golden that lists them.
