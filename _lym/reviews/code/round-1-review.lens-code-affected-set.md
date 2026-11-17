```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens for round 1, covering what the diff forces outside the lines it touches.

**Per change, the affected set found, covered or not**

1. **`isReadOnlyLookup` moved to `bashlookup.IsReadOnlyLookup`, and `LineRangeFile` added.**
   - Covered: callers `claudeengine/batchingnudge.go:249,360` and `codexengine/batchingnudge.go:201,426`; test `bashlookup_test.go`; the doc mention at `codexengine/doc.go:128`; the package list at `docs/overview.md:639`.
   - Covered: the `TestMain` that `PATTERN-test-isolation` requires (`bashlookup/testmain_test.go`).
   - A grep finds no stale `isReadOnlyLookup` or `claudeengine/bashlookup` reference.
2. **`readTranscriptTail` and `countLinesUpTo` moved to `shuttleengine.ReadFileTail` and `CountLinesUpTo`.**
   - Covered: the callers in both engines and the moved test in `filetail_test.go`. No stale reference.
3. **Seam-scan exemption (`sharedPackages`).**
   - Covered: the `PATTERN-shuttle-provider-seam.md:10-11` background, and a test row in `seam_enforcement_test.go`.
   - Not covered, see F3: the commit message overstates the exemption.
4. **`hookSettings` signature changed to take `toolHooks`.**
   - Covered: the one production caller (`codexengine.go:259`), `TestHookSettings`, and every `prepare_test.go` call site.
   - `OpenLaunch` does not use `hookSettings`, so `docs/overview.md:644` ("opened Codex session … no shell guard and no creel hooks") stays true.
5. **`denyNotice` signature changed to `(questionDeny, rows)`.**
   - Covered: the caller `codexengine.go:168`, `guards_test.go`, `prepare_test.go` `wantCreelHead`, and `codexengine/doc.go:103-107`.
6. **Question-tool `PreToolUse` entry on the Codex launch line.**
   - Covered: `codexengine/doc.go`.
   - Not covered, see F1: `docs/overview.md:713`.
7. **Codex `batching_nudge` block, the hook entry and `BatchingNudger` on Codex.**
   - Covered: `config.go`, `template.yaml`, `codexengine/doc.go:109-144`, `TestLoadConfig_Template`, `TestLoadConfig_RefusesInvalidValues`, `TestBatchingNudger`, and `shuttleengines/doc.go:34`, which is engine-generic.
   - Not covered, see F1: `docs/overview.md:705/713`. See also F2: the `shuttlecli/hookbatching.go` help text.
8. **`rollout.go` split into `parseRolloutEvents` and `latestContext`.**
   - Covered: the `ContextTokens` caller and the codex nudge caller. Behaviour is unchanged.
9. **Declared PATTERN changes:** the directive declares none, so this raises nothing. The background edit in item 3 is consistent with the `PATTERN.md` index line.
10. **Commit messages:** no `Pattern-Departure:` trailers. See F3.
11. **Change description (`_lym/landing/summary.md`):** every claim I checked against the affected set holds. That covers the `{}` fallback, the hook order, the moved tests and the `rollout.go` split.

**Findings**

- **F1: LOW, consistency, `docs/overview.md:705` and `docs/overview.md:713`.** The overview's shuttle section still describes the batching nudge and the question-tool deny as Claude-only.
  - Line 705 names only `claude.yaml`'s `batching_nudge`.
  - Line 713 says a Codex launch installs the shell guard rows as `PreToolUse` hooks matching `Bash`. It does not mention the `*`-matcher `request_user_input` deny on an autonomous launch, nor `codex.yaml`'s `batching_nudge` `PostToolUse` hook that the diff adds.
  - The diff touched this same file, at line 639, for the package list only.
  - Evidence: `docs/overview.md:705,713`; `codexengine/command.go:278-289`.

- **F2: NIT, consistency, `internal/shuttlecli/hookbatching.go:26-32`.** The verb's help text still describes the judgment in Claude's terms. It says the verb "judges the call and the calling chain's recent messages", while the Codex engine now served by this verb judges a thread's exec cells keyed by `transcript_path`.
  - The text stays roughly true, but it no longer names what one of its two engines does.
  - Evidence: `hookbatching.go:28`; `codexengine/batchingnudge.go:387-393`.

- **F3: NIT, consistency, commit `f381f428` message.** It says the scan "exempts packages under internal/shuttleengine that hold no engine". The code exempts only the packages named in the explicit `sharedPackages` list (`bashlookup`), so a new engine-free package is not exempted automatically. The pattern background states it correctly ("the scan's `sharedPackages` names each one").
  - Evidence: `seam_enforcement_test.go`, the `sharedPackages` var; `git log` body of `f381f428`.

**Tool calls**

1. A grep across `.go`, `.md` and `.yaml` for the nudge, question-deny and `bashlookup` terms, plus `shuttleengines/doc.go:28-40` and the doc mentions in both engines. My context lacked a repo-wide search of the statements that describe the changed behaviours.
