```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff, one case per function or branch. No other lens's findings are included.

## Per-function list

- `keyedEngine`, `passagesOf`, `Passages` and `FileEntries` (`internal/heddleengine/render.go`). I ran an entry aimed at `agents-prompt` plus `claude-prompt`, an entry aimed only at `claude-prompt`, and an entry aimed at `orch-prompt`.
  - Entries aimed at only one engine's target are keyed to that engine, and any shared or engine-neutral target gives `""`.
  - The empty and unknown engines return nil.
  - Correct.
- `Target.Engine` newly set on `claude-file` and `codex-file` (`internal/heddleengine/target.go`).
  - The other reads of the field are `promptTargetsFor`, which filters on `Delivery == DeliveryPrompt` first, and `promptReach`, which is called for prompt targets only.
  - No behaviour change. Correct.
- `Show` (`internal/heddleengine/render.go`). Passing `engine=""` only affects `Passage.Engine`, and only `.Text` is used. Correct.
- The passage reader in `cmd/lym/main.go`. A failure in `FileEntries` drops the passages too, but `StandingPassages` already treats a reader error as zero `Standing` with a Warn. Correct.
- `copyManifest` and the `launch` hook in `internal/shuttleengine/run.go`, covering these cases:
  - Record absent: writes nothing.
  - Record undecodable: error, only logged.
  - `manifests` path blocked by a file: error, only logged.
  - Normal case: the copy keeps the written verdict.
  - Correct.
- `findRunByStrand` and `sweepOrphans` skip the `manifests` directory. Correct.
- The other root-directory readers (`agentwait`, `activity`, `sessionread`, `waitmark`, `attach`, `stepstop`, `carryover`) don't skip it.
  - Each calls `loadRunState`, finds no `run.json` and skips, so they are correct.
  - Side effect: `state.ReadJSON` creates a `.lock` file inside `manifests/`, which is harmless.
- `claudeManifest` and `manifestInputsFor`.
  - Each `artifact` case reads the intended file.
  - An empty anchor makes `EvalSymlinks("")` return `"."`, which is correct enough.
  - A skill-load proof that is the same for every skill is correct for one skill. I could not confirm the `", "` joiner against `SkillLoadMessage`.
  - Incorrect: `resolvedCandidates` (see F1).
- `claudeengine/Prepare`: the manifest write failure refuses the launch. Correct.
- `codexManifest`, `holds`, `lineHolds` and `tomlContent`.
  - Shell and TOML escaping is per character, so the inner-substring check holds, and the zero-length quote can't occur.
  - Correct.
- `setupCandidates` in `codexengine/manifest.go`. A hook with an empty `Plugin` is skipped (see F2).
- `claudeFileCandidates`: an empty `found.root` makes `filepath.Rel("", …)` error, so the file is skipped. Acceptable.
- `resolveUserSetup`. `Reason == ErrHookUnrun.Error()` matches, because `RunHook: c.runHook` is passed through unwrapped (`codexengine.go:211`).
- `Compare`.
  - It drops `webster-fork-deny` unless Codex supports `RequirementForks`, which gives the required tripwire.
  - A missing reference manifest silently yields an empty result, but `PrepareEach` guarantees both manifests exist.
  - Correct.
- `PrepareEach`: every error names the engine, and the skill expansion mirrors a launch. Correct.
- `Report`: the temporary directory is removed by `defer` on every path. A builder failure costs only that builder's seats. Correct.
- `specsOf` and `seatengine.Specs`: model choice reduced to the engine alone, prompts as `Prompts` renders them. Correct.
- `buildBurler`, `FixReviewSeatTable` and `SeatTables`. The builders only compute paths and stand-ins. I did not verify the "reads nothing" comment beyond that.
- `manifestFields` and `manifestCmd`.
  - `missing` is an empty slice, so it prints `[]`.
  - The `Verdict` copy is edited on a value, so the report is not mutated.
  - Non-zero exit with each missing item named. Correct.
- `manifestReportOf`: refuses when the hub has no shortname, and names the way forward. Correct.
- `NewWithSeams`, `registeredConstructors` and `build`: `New` and `keepWarmWith` pass the zero seams, and a nil `RunHook` falls back to `RunPluginHook`. Correct.
- `UnrunHooks` returns `ErrHookUnrun` unwrapped. `ConfiguredSandbox` returns nil. Correct.

## Findings

**F1 — MEDIUM, design.** `internal/shuttleengine/claudeengine/manifest.go:331-333`
- `resolvedCandidates` swallows a `UserSetup` error with a Warn and carries on with the zero `setup`.
- Claude is the reference manifest, so every instruction file, skill listing and plugin context it fails to resolve is silently dropped from the comparison.
- The parity test then passes with fewer compared items; nothing is named as missing. The comment's claim that "the comparison names what the manifest then lacks" is false for the reference engine.
- Fix direction: return the error from `manifestInputsFor` so `Prepare` or the test fails loudly, or record an omitted item that Codex's manifest can be held to.

**F2 — LOW, design.** `internal/shuttleengine/codexengine/manifest.go:300-303` against `claudeengine/manifest.go:345-349`
- When a plugin's hooks file is unreadable, `OmittedHook.Plugin` is empty.
- Claude lists that hook as an omitted `plugin-context` item keyed by `hook.Name`.
- Codex skips it with `if hook.Plugin == ""`.
- `MissingItems` then reports a Codex lack for what is one shared setup fault, not a dropped Codex delivery.
- Fix direction: list it on Codex under the same `hook.Name` key as an omitted item.

## Tool calls
1. `grep` for `ReadDir`, `isManifestsDir`, `dirs.Anchor`, `EvalSymlinks` and `runHook`, to find the other root-directory readers, the Claude anchor dependency and the Codex hook wrapping. The context lacked those definitions.
2. `grep -A` and `sed` over the readers in `activity.go`, `stepstop.go`, `attach.go` and `agentwait.go`, to see how each handles a directory with no `run.json`. The context lacked those definitions.
3. `grep` for `.Engine` in `heddleengine`, `skillLoadMessage` in `claudeengine`, and `promptTargetsFor`, to check other readers of the newly set file-target `Engine` field. The context lacked those definitions.
4. `sed` of `internal/heddleengine/overview.go` lines 135-165, to confirm `promptReach` runs for prompt targets only. The context lacked that code.
