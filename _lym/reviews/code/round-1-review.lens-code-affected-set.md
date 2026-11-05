```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, I list what reads, asserts or documents it, and whether the diff covers it.

**Per-change list**

1. **`writable_roots` key (`config.go` `Config`, `template.yaml`, `ConfigSpec` with `CategoryGuard`)**
   - Set found:
     - `configreg_test.go`'s `TestModules_EveryTemplateKeyClassified`, which carries the `LocalDenied` record. Covered (`codex: {"binary","writable_roots"}`).
     - `TestLoadConfig_Template`. Covered.
     - `PATTERN-config-layers`, which says every non-None category closes the key to the local layer and a new key is classified in the same change. Covered.
     - The `codex.yaml` template comment. Covered.
     - No other doc lists the `codex.yaml` keys (the grep for `skill_roots` found only `doc.go` and `skillload.go`). Covered.
   - Finding: none.

2. **Git dirs added as writable roots (`readCheckout`, `checkout.gitDirs`, `readyHome`)**
   - Callers of `readCheckout`: `readyHome` and `CheckAfterStartup`.
   - `CheckAfterStartup` now also fails its post-startup check if `EvalSymlinks` of a git dir fails.
     - It only warns, and `FindRoot` has already checked that the dirs exist, so this is not a finding.
   - Callers of `readyHome`: `Prepare` and `OpenLaunch`. Both pass the roots on through `launchLine`.
   - Tests that assert the line:
     - `TestPrepare_Launch`, `TestPrepare_InstallsGuardsFromItsInputs` and `TestOpenLaunch` use `wantGitDirs`. Covered.
     - `command_test.go` builds `launchLine` directly, so it is unaffected.
   - Docs:
     - The doc comments on `Prepare`, `readyHome` and `doc.go`, plus the shuttle row of `docs/overview.md`. Covered.
     - `OpenLaunch`'s doc comment does not say that an opened session also gets the git dirs and configured roots.
       - It says "readies the same home as Prepare does", so this is not a finding.
   - Finding: none.

3. **Removal of the "writable-roots setting only when the records checkout is told" assertion**
   - The `DeepEqual` on the whole launch replaces it.
   - Finding: none. The git dirs are now always present, so the old assertion no longer held.

4. **Research docs under `docs/research/*` that state the old launch line (records checkout as the only writable root)**
   - They are dated trial and bench reports, i.e. measurement records.
   - Finding: none, since the rubric exempts a measurement report.

5. **`PATTERN.md` and `pattern/` changes**
   - The directive lists no declared PATTERN changes, so none were required.
   - `PATTERN-shuttle-provider-seam` is respected: the changes sit under `internal/shuttleengine/codexengine`.
   - `PATTERN-no-hard-coded-choice`: the `~/.cache/go-build` default is a template value, not a code branch. No finding.
   - `PATTERN-lym-temp-root` governs lym's own temp files. The entry's `GOCACHE` example does not fall under it, because the diff creates no temp file.
   - `PATTERN-refusal-way-forward` binds only webster, shed and loom, not `codexengine`. The `EvalSymlinks` non-`ErrNotExist` refusal in `configuredWritableRoots` carries no `way forward:` clause. Not a rule violation.
   - No finding.

6. **Commit messages**
   - No `Pattern-Departure:` trailer appears in any of the three commits.
   - Finding: none.

7. **Comment conventions in `docs/overview.md` and the code comments**
   - Line wrapping is not a finding.
   - Finding: none.

**Findings:** none from this lens.

**Tool calls (1 of 6)**
- `grep` for `workspace-write`, `records checkout it is told` and `writable root` across `.md`, `.yaml` and `.go` files. The manifest had checked only `skill_roots` for docs listing the codex keys, so it lacked other documents describing the sandbox's writable roots. The result was only `docs/overview.md`, the `codexengine` files and dated `docs/research` reports.
