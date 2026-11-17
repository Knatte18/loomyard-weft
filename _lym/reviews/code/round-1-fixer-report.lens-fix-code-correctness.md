Task: check the correctness of the code changed by the fixes for F2–F10 (the fix-code-correctness lens).

**Findings**

- **LOW** — `internal/shuttleengine/claudeengine/subagents_test.go`, keep mark on `TestPrepare_EffortFromAgentsConfig`.
  - The mark says the covering tests "do not assert" the `--effort` value per model resolution.
  - `TestPrepare_ThreadsSpecIntoLaunchCmds` asserts `--effort 'high'` for an explicit effort (`prepare_test.go:368-369`).
  - The mark is still right for the agents-config-derived rows, because the covering test never loads an agents config.
  - The wording should narrow to "per model alias or full model ID resolved through the agents config, and the spec-effort-wins rule". As written it overclaims.

Everything else holds:
- **F3:**
  - `EngineName` now returns `(string, error)`, so every `TidyShuttle` implementer and caller compiles; the gate passed for `dobbyengine` and `beamcli`.
  - On the error path `spawnTidy` returns before `Start` and before `UpdateState`.
  - The new subtest fails with the defect in place: ignoring the error would start an agent.
- **F5:** every `driverPrompt` reference is renamed, the tests compile, and the gate passed.
- **F7:** with the pre-fix lookup, a Codex default would miss the claude engine, so the new row would fail.
- **F8:** both assertions fail if `PromptEngine` is dropped from the spec.
- **F9:** both rungs-with-engine assertions fail if the code reads `Ladder[0]`.
- **F4:** `RequireEditDirectiveOf` fails if a producer renders the wrong copy, since the claude and codex copies share no substring. The seatengine edit was reverted, and the gate was green for it afterwards.

**Tool calls**
- `grep` for `--effort`, `advisorModel` and `SUBAGENT_EFFORT` in `claudeengine/prepare_test.go`: to check whether the covering tests assert what the F10 keep marks claim; my context lacked those test bodies.
