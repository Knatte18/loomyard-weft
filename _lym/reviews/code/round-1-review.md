---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/loomcli/wiring.go:675
    summary: the engine name DriftBaseline is a bare string literal in wire, where the neighbouring engine names are named constants
    fix: In internal/loomcli (next to darnWriteEngine in arm.go) declare a named constant for the DriftBaseline engine name, or reuse one if the shedrecipe package exports it, and pass that constant to loomrecipe.HasEngine at wiring.go:675 in place of the literal.
    origin: handler
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/driftengine/diffmode.go:305
    summary: skipFileArgument consumes the closing quote of an enclosing sh -c string, so the migration writes an unbalanced command and Preflight no longer finds the leftover
    fix: In skipFileArgument, consume a trailing quote only when the assignment's opening quote was matched (a capture group for the opening quote and a backreference-free check in Go, such as two alternatives, quoted and unquoted). Add a row to TestMigrateConfig_SkipFileArgument for `sh -c 'docker run -e A={skip_file}'` that expects the command unchanged or correctly closed.
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/driftengine/diffmode.go:305
    summary: removing an argument from a multi-line command with backslash continuations leaves a stray escaped space and silently changes the command
    fix: In withoutSkipFileArguments, when the matched leading whitespace holds a newline, replace the match so the preceding backslash and line break stay (or, for a match the regex cannot rewrite safely, leave the command alone so FlagDiffMode and CheckRetiredPlaceholders name it). Add a TestMigrateConfig_SkipFileArgument row with a `\`-continued multi-line command that expects the remaining lines intact.
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: design
    location: internal/driftengine/verify_test.go
    summary: no test fails if openJudgeSession opens the session mark before taking the snapshot
    fix: Add a row to the drift verify test that makes the snapshot fail (a Checkouts error) on a fresh judge session and asserts the judge_session mark is not open afterwards.
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: design
    location: internal/driftengine/config_test.go
    summary: no test pins that volatile_files and volatile_keys are refused in a task's run settings with the hub's drift.yaml as the way forward
    fix: Add a row to the run-settings guard-key test covering volatile_files and volatile_keys beside diff and key_patterns, asserting the refusal names the hub's drift.yaml.
    origin: lens:code-tests
  - id: F6
    severity: LOW
    class: design
    location: internal/driftengine/expect.go:125
    summary: DriftExpect.pass records scope.Volatile into a scope record that holds no diff mode, and no test covers it
    fix: Add a DriftExpect test row in which the scope record holds no diff mode and the config sets volatile lists, and assert the recorded scope holds them.
    origin: lens:code-tests
---

### [BLOCKING] DriftBaseline engine name is a bare literal

**Location:** internal/loomcli/wiring.go:675
**Issue:** `loomrecipe.HasEngine(c.recipe, "DriftBaseline")` hard-codes a registry row name, which `PATTERN-code-conventions` requires to be one named constant ("a name or path that can go stale is one named constant"). The neighbouring `darnWriteEngine` in `arm.go:334` is a constant. The row name also appears in `shedrecipe/registry.go` and `entries_drift.go`, so a rename would leave this check silently false and skip the Preflight drift check.

### [MEDIUM] Migration eats the closing quote of an enclosing shell string

**Location:** internal/driftengine/diffmode.go:305
**Issue:** The regex's trailing `["']?(\s|$)` consumes a quote before testing for the end of the argument. For `sh -c 'docker run -e A={skip_file}'` the match includes the quote that closes the `sh -c` string, so the migration reports success and writes an unbalanced command. Preflight then no longer finds a leftover, so the failure moves to run time. The tests cover single-line commands only.

### [MEDIUM] Migration breaks backslash-continued multi-line commands

**Location:** internal/driftengine/diffmode.go:305
**Issue:** The leading `\s+` of the match starts at the newline after a `\` and one whitespace character is kept. `img \⏎  -e A={skip_file} \⏎  -v …` becomes `img \ \⏎  -v …`: the `\ ` is an escaped space and passes a literal space argument to the container command. Hub commands written as YAML block scalars make this form realistic.

### [LOW] Open-mark-has-snapshot ordering is unpinned

**Location:** internal/driftengine/verify_test.go
**Issue:** The reordering in `openJudgeSession` (snapshot, then mark) is the fix for a mark that outlives a failed snapshot. No row fails if the order is reverted.

### [LOW] Hub-only refusal of the volatile keys is untested

**Location:** internal/driftengine/config_test.go
**Issue:** The `CategoryGuard` and `HubOnlyWayForward` entries for `volatile_files` and `volatile_keys` have no test row, though the decision record asks for a hub-wide list.

### [LOW] Expect-pass recording of the volatile lists is untested

**Location:** internal/driftengine/expect.go:125
**Issue:** Only the `recordDiffMode` copy in verify is covered; the second writer in the expect pass is not.

## Rejected

- Lens code-affected-set F1 (`MigrateConfig` instead of a reinterpretation, against `PATTERN-config-layers`): the module's existing `FlagDiffMode` reinterpretation still ships and still flags the placeholder; the migration is an addition that the change description justifies, so the entry's requirement is met.
- Lens code-correctness F3 (`CheckRetiredPlaceholders` fails on any config load fault): intended, since the same fault would stop the run at its first Drift row, and the change description says the load check is kept.
- Lens code-tests F4 (nil `PreflightDrift` for a recipe with no Drift row): trivial branch, `envkit` marks the field nil-legal; a test would assert the absence of wiring.
- Lens code-tests F5 (empty `ScratchDir` text and `Probe` error path): error-path and text variants of branches the existing rows reach; no behaviour is left unpinned.

Holistic checks with no finding: the order in `resumeJudgeSession` (clear, then snapshot), the exemption of `scope.json` and `judged.json` only while both match their newest `Drift-Records` commit (a hand edit still halts), the halt text that keeps the snapshot directory, the snapshot-gone path under a live chair, the atomic record replacement, the hub-wide volatile keys and their template documentation, and the change description's statement of what the Models hub must set all match the decision record.
