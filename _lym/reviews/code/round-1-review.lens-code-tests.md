```yaml
lens: code-tests
outcome: reported
```

**List per behaviour (code-tests lens)**

| Behaviour | Pinning test |
|---|---|
| `SteerFor` and `NoticeFor` return the Codex text, or the plain text for an engine without one | `TestRows_EngineTexts` (`rows_test.go:109`), including `SteerFor("claude")` and `NoticeFor("claude")` equal to the plain text |
| The Codex texts name no Claude tool, and the Codex edit rule is stated in exactly one notice | `TestRows_EngineTexts` |
| `checkTexts` refuses a forbidden character in an engine text | `TestCheckTexts`, rows `EngineSteerSingleQuote` and `EngineNoticeLineBreak` |
| Claude's render is unchanged | `TestStandingDenies_RenderEveryShellGuardRow` (`settings_test.go:434`). It reads `row.Steer` and `row.Notice` directly and the diff doesn't touch it. |
| `installedGuards` covers every row in table order, derived from `shellguard.Rows` with no literal count, and follows the python, publish and refuse-list inputs | `TestInstalledGuards_CoverEveryRow` (`guards_test.go:28`). An unknown switch would panic inside this test, so a new switch fails it. |
| One Bash entry per row, in order, with the Codex steer in the deny | `TestGuardEntries_RenderEveryRowWithItsCodexText`. This is partly self-referential; see finding 1. |
| The deny notice joins the Codex notices, and `denyNotice(nil)` is `""` | The same test |
| Guard commands run under `sh`: deny on a match, `{}` otherwise, all patterns required, quoted patterns | `TestGuardCommand_DeniesAMatchingCommand` (tier 1). It runs `machine-sleep`, `shell-edit`, `computed-rm`, `refuse-raw` and `publish`. `computed-rm` is the two-pattern row. |
| `hookSettings` puts all guard entries in one `PreToolUse` setting, ordered recorded → guards → activity, with none when there are no guards | `TestHookSettings` |
| The prompt file opens with the deny notice, then the passages, then the prompt | `TestPrepare_Launch` and `TestPrepare_AppendsStandingPassages` |
| Prepare installs the python row from `cfg.DenyPython` and the publish row from `spec.DenyPublish` | `TestPrepare_Launch` sets both to true; see finding 2 |
| A spec with `DenyPublish` is no longer refused | The `PublishGuard` refusal row was removed, and `TestPrepare_Launch` now launches with `DenyPublish: true` |
| The hub gate config's `refuse_raw` list reaches the installed rows | None; see finding 3 |
| A gate config that exists but fails to load refuses the launch | None; see finding 3 |
| `skillPath` picks the newest version by number across the first root that holds the skill, returns none for an absent skill or one with no plugin | `TestSkillPath` (`1.10.0` beats `1.9.0`; two roots) |
| `SkillLoadMessage` names found skills in order, leaves an unfound one out, and names no file when none is found | `TestSkillLoadMessage` |
| `ClassifySkillLoad` reads the load turn from the rollout | `TestClassifySkillLoad`. Covered: a call after the message is loaded, a call before it is missing, an unfound skill is unknown, a rollout with no matching message is unverified, an absent rollout is unverified, both user-message shapes. |
| A spec with skills launches with the load line as its first turn and the prompt pointer as `PromptLine` | `TestPrepare_SkillsLoadFirst` |
| `compareVersions` text, prefix and mixed-field branches | None; see finding 4 |
| `skillRoots`: `~/` expansion, an unresolvable root dropped with a Warn, a `codex.yaml` that fails to load leaves no roots with a Warn | None; see finding 5 |
| `loadTurnCalls`: `custom_tool_call` and `local_shell_call`, a repeated load message resetting the calls, garbage or truncated lines | None; see finding 6 |
| `DefaultSkillLoadTimeout` | None. It is a constant accessor and I don't count it. |

**Findings**

1. **LOW, design.** `guards_test.go:68` and `guards_run_test.go:52` build the expected deny from `denyDecision()` itself. No test fixes the spike's deny shape, so a change to the JSON keys passes every test. Pin the literal in one assertion, or check that the output parses as the `PreToolUse` deny shape.

2. **MEDIUM, design.** The wiring in `codexengine.go:92` has no test that tells the two inputs apart. `TestPrepare_Launch` sets `DenyPython` and `DenyPublish` both true, and `TestPrepare_AppendsStandingPassages` sets both false. A swap of `cfg.DenyPython` and `spec.DenyPublish` passes both. A row with only one switch on would fail it.

3. **MEDIUM, design.** No test drives `Prepare` with a hub `gate.yaml`. The grep over the codexengine tests found no `gateslot` or `refuse_raw` use in `Prepare` tests. The hub's `refuse_raw` list reaching the refuse-raw hook and the notice is unpinned. So is the refusal when the gate config exists but fails to load, which the summary and the `Prepare` doc claim. `TestPrepare_Launch` passes `nil` to `installedGuards`. The behaviour is new in this diff at `codexengine.go:88-91`.

4. **LOW, design.** `compareVersions` (`skillload.go:140`) has no row for the text-comparison fallback, a non-numeric or odd version directory, or the prefix ordering. `TestSkillPath` covers only numeric dotted versions.

5. **LOW, design.** `skillRoots` (`skillload.go:84`) has no test for `~/` expansion, which is the default `~/.claude/plugins/cache`. It also has none for a dropped root with a Warn, or for a `codex.yaml` that fails to load. `skillEngine` always passes an absolute root.

6. **LOW, design.** `loadTurnCalls` (`skillload.go:175`) has no test for the `custom_tool_call` and `local_shell_call` payload types, only `function_call`. It also has none for a second load message resetting the collected calls, or for a truncated line. The first two are named in the focus failure paths.

Test economy: no duplicate or per-helper test found. `TestRows_EngineTexts` and `TestCheckTexts` assert different things. `TestGuardEntries_RenderEveryRowWithItsCodexText` partly restates the implementation (finding 1).

Tool calls: one `grep` of the codexengine `_test.go` files for `gateslot`, `gate.yaml` and `refuse_raw`, because the manifest showed only the diff hunks of `prepare_test.go`.
