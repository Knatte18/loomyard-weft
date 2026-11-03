```yaml
lens: code-tests
outcome: reported
```

Lens `code-tests`: for each behaviour the diff adds or changes, the test row that pins it, or none. I read `prepare_test.go` and the fixture, and took the other test files from the name grep. I did not read their bodies.

**Pinned**

| Behaviour | Test |
|---|---|
| Launch line and resume line, with the weft writable root, model and effort | `TestBuildLaunchCmd`, `TestPrepare_Launch` (`prepare_test.go:40`) |
| Hook settings and TOML quoting | `TestHookSettings`, `TestTOMLString` |
| Hook commands, run under `sh`, record and stamp | `TestHookCommands_RecordAndStamp` |
| Writable root is the parent of the `.lym` and `_lym` link target | `TestPrepare_Launch` (`prepare_test.go:60`) |
| Prompt file opens with the standing passages | `TestPrepare_AppendsStandingPassages` |
| Home links, trust of the worktree and main roots, vanished root dropped, unknown record key kept, `config.toml` rendered | `TestPrepare_ReadiesHome`, `TestRenderConfigTOML` |
| Concurrent launches lose no root | `TestPrepare_ConcurrentLaunchesLoseNoRoot` |
| Refusals write nothing (`prompt.md` and the home are asserted absent): version pin, permission mode, resumed session, publish guard, effort not in `efforts`, `.codex/rules` in the worktree | `TestPrepare_RefusesBeforeWriting` |
| Warns: worktree config, main checkout config behind a worktree `.codex` folder, regular `auth.json`, `sessions` linked elsewhere | `TestPrepare_Warnings` |
| `StartupChecker` call in `run.go` | `TestStartup_ChecksAfterStartup` |
| `CheckAfterStartup` warns on a lost root and stays silent otherwise | `TestCheckAfterStartup` |
| `codex.yaml` template and `~/` expansion | `TestLoadConfig_Template`, `TestExpandHomePath` |
| `ParseEvents` and `ParseSessionSignals` | `TestParseEvents`, `TestParseSessionSignals` |
| Rollout interrupt and context tokens | `TestTurnStartInterrupt`, `TestContextTokens` |
| Pane classification, trust dismissal, composer readings, paste placeholder, key sequences, `PasteSubmits` | `pane_test.go` |
| `CreateFileLink` on Linux: in-place write reaches the target, clobber refused, parent created | `TestCreateFileLink` |
| `CreateFileLink` on Windows: refusal and parent left uncreated | `TestCreateFileLink`, which asserts it only when run on Windows |
| Built-in aliases, template aliases, `knownEngines`, spec text | `load_test`, `registry_test`, `template_test` |
| `CODEX_` scrub | `TestEnvScrub_MatchesTmuxkitServerEnviron` (now carries `CODEX_HOME`) |
| Registry names | `TestNew` |

**Findings**

1. **LOW, design.** No test covers the engine's behaviour on an anchor whose `.lym` and `_lym` are real directories, which is the outside-a-hub case where the `writableRoots` helper returns none (`home.go:117`).
   - `fixture_test.go:addWorktree` always symlinks them.
   - The `len(writableRoots) > 0` branch in `buildLaunchCmd` is covered only by `TestBuildLaunchCmd` row `NoWritableRootModelOrEffort`, not through `Prepare`.
   - The Sketch's D6 and the summary both state this behaviour, so a regression would pass.

2. **LOW, design.** Prepare's trust handling has no test for a plain clone, where `mainRoot` is empty, or for `mainRoot == root`.
   - Every fixture is a linked worktree (`fixture_test.go:addWorktree`).
   - The `trustRoots` one-root branch and the `mainRoot != resolvedRoot` guard at `home.go:72` are therefore unpinned.

3. **LOW, design.** The rules-refusal test covers only the worktree root.
   - The second `folders` entry, where the resolved anchor differs from the worktree root, has no row (`home.go:85`).
   - `TestPrepare_RefusesBeforeWriting` `CommittedRules` creates `.codex/rules` at `anchor`, which is the root.

4. **LOW, design.** Prepare's two error paths have no test (`codexengine.go:79` and `codexengine.go:91`).
   - One is an invalid `codex.yaml`, which should carry the way forward from `loadConfigOrRefuse`.
   - The other is an empty or relative anchor.
   - `TestLoadConfig_Template` and `TestExpandHomePath` cover neither path.

5. **LOW, design.** A configured `binary` reaching the launch line is unpinned (`config.go:80`).
   - `TestPrepare_Launch` only sees the default `"codex"`.

6. **LOW, design.** `CheckAfterStartup`'s failure paths have no test (`home.go:222-241`).
   - `TestCheckAfterStartup` covers only the lost-root and silent cases, not a config that fails to load, a non-repository anchor or an unreadable `config.toml`.

7. **LOW, design.** `ensureLink`'s "already points at target" path asserts no warning on a second Prepare.
   - Only the concurrent test exercises a re-run, and it checks no warnings (`prepare_test.go:163`).
   - A regression to always warn would pass.

8. **LOW, design.** Constant-returning methods have no row: `PromptCacheLifetime`, `KeepWarmCaps`, `ProcessLiveness`, `CompactedSince`, `InterruptSettle`, `SubmitSettle`.
   - `InterruptSettle`'s 2-second contract (D8) is the only one a requirement names.
   - The grep shows no test referencing it.

9. **LOW, design.** The Windows refusal in `CreateFileLink` and Prepare's ordering are asserted only on Windows.
   - `TestCreateFileLink` returns early on Linux.
   - No test pins that `prepareHome` links `auth.json` before any home write, which "refuses before anything is written" relies on (`home.go:132-140`).
   - Nothing on Linux fails if that ordering breaks.

I did not judge the Added-after-Discussion writable-root proof (`codex sandbox`) as a test gap; it is a build-time check, not a test.

Tool calls:
1. Bash grep of test function and case names across `codexengine/*_test.go`, because my context held none of the test files.
2. Read `prepare_test.go` lines 1-365 for the Prepare and home assertions.
3. Bash grep for the trivial methods plus the head of `fixture_test.go`, to see whether the fixture ever uses real `.lym` directories.
