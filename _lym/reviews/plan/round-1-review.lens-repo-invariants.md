```yaml
lens: repo-invariants
outcome: reported
```

**Lens repo-invariants: repository invariants per card, with one finding (card 9 doc).**

| Card | Test seams and tier tags | Docs falsified | Passes own gate from prior state |
|---|---|---|---|
| 1 | Held. `hubregistry_test.go` is tier0 and `testmain_test.go` is untagged. `TestUserStateDir` is declared non-parallel. | Held. `PATTERN-test-isolation` and `standalonestate/doc.go` are listed. `PATTERN-lym-files` says every never-tracked file is under `.lym`, but the record's Constraints exempt `hubs.json`, and card 1 cites the precedent. | Held. Tests that set `HOME` read `.claude` or `.tmux.conf`, not `.local/state`. Windows goes through `setTempEnv` too (`tmuxkit.go:91`). `TestDeriveCallerSet_CliwireOnly` matches `Derive` only, so `UserStateDir` passes. |
| 2 | Held. `config_test.go` is tier0 and the testmain is untagged. | Held. The config-layers list and overview are listed. | Held. The strictness set changes in the same card. `TestNames` is generic, and the only pinned hub-wide list, `TestModules_SeedOnlyAndHubWideFlags`, is listed. |
| 3 | Held. No reflective test walks `Message`, `record` or `LogEntry`: no `NumField` and no `lym:seam internal/creel`. `TestWriteMessagesText` is tier0 and `TestCreelCLI` is tier1, as the card says. | Held. Both `doc.go` files are listed. | Held. |
| 4 | Held. `hubdirectory_test.go` is tier0 and uses no hubforge. `clientSeam` is not in any seam marker. | Held. | Held. The only `clientSeam` test literal is `cli_test.go:299`, as the ImpactSummary says. |
| 5 | Held. `rig.addHub` sits in the tier1 file `cli_test.go`, so `CopyHub` is allowed. | Held. | Held. `AudienceRole` has no test requiring the role prompt to name the verb, so card 10 can come later. |
| 6 | Held. | Held. | Held for card order. I did not check whether an existing test asserts the old refusal for a name of another hub. The ImpactSummary's "which no test asserted" stays unconfirmed. |
| 7 | Held. Every `creelengine_test` file is tier0, so the unnamed test file takes one tier. `Request` and `Response` are not walked reflectively. | Held. | Held. `unpark` has nothing to park until card 8, which only leaves a helper unused. |
| 8 | Held. | Held. `PATTERN-daemon-wakeups` and `unboundedWaitFiles` change in the same card. | Held. |
| 9 | Held. | **Broken: `internal/reportengine/doc.go`, see the finding below.** | Held. |
| 10 | Held. Only `entries_bouncer_test.go:1189` names the directive, and it writes its own stencil, so nothing pins the prose. | Held. The shortened `PATTERN.md` line keeps `(test)` and its link. | Held. No fixture or copy of the trust sentence exists outside the constant. |

**Finding 1**
- **Severity:** MEDIUM
- **Class:** consistency
- **Location:** `_lym/plan/09-replies-across-hubs.md`, `**Uses:**` line 27 and Intent lines 46-47
- **Summary:** Card 9 changes what `ComputeReplies` counts, but lists `internal/reportengine/doc.go` only under `**Uses:**`, so nothing updates its `# Replies` section.
  - The card makes `inbound` read as `posted`, and makes an `outbound` entry whose `Answers` names a marked message count `Replied`.
  - The doc still defines Expected as "the `posted` entries" and Replied as "the `posted` reply entries".
- **Evidence:** `internal/reportengine/doc.go:205-207`, and `internal/reportengine/replies.go:48` for the current `posted` case.
- **Fix:** move `internal/reportengine/doc.go` from `**Uses:**` to `**Edit:**` on card 9, and state the new counting in the Intent. This target move has not been run through `validate-plan --overlay`.

**Tool calls (6)**
1. Seam markers, `seam_enforcement_test.go`, headroom lint sites and per-file tier counts: the manifest had no test-tier or seam data.
2. Readers of `XDG_STATE_HOME` and tests that set `HOME`: needed to judge card 1's `setTempEnv` change.
3. Config module list sites, trust sentence copies, cross-hub doc statements, stencil hash code and verb lists: needed for cards 2 and 10, and these sites were outside the manifest.
4. Reflective tests, `callerset_enforcement_test.go`, `HOME` tests without XDG, `TestNames`, and creel mentions in other pattern files: needed to confirm seams and caller pins.
5. `setTempEnv` callers, `clientSeam` literals, import pins, `reportengine/doc.go`, pattern check kinds and pinned prompt text: one hop past the manifest.
6. The bouncer test pin, the `tmuxkit.Main` body and the `AudienceRole` rule: needed to settle the card 5 and card 10 order checks.
