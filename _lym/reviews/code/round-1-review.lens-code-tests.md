```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test that pins it, or none.

**Per behaviour → pinning test**

- Trust sentence, flag dir and flag-name fragment moved to `shuttleengine`, Claude unchanged → `claudeengine` `checkOpenCreelFiles` and `TestPrepare_CreelMailHookFollowsTheBoardDirectory`; `hubgeom` `TestCreelGeometry`. Pinned.
- `StrandFlagNameShell` run under `sh` → `TestCreelHookCommands` (the mail and Stop rows with and without the flag). Pinned.
- `IdleWaker` dispatch in `wakeSession` → `TestWakeSession`, rows `WokenByTheHookExit`, `WakesItself` and `WakeFails`. Pinned. The failure path leaves the mail unread because `hookWait`'s `writeWake` returns the error before `wroteLines` is set, and `hookWaitExit` returns 0 when `wroteLines` is false. That second half is pinned by the existing `TestHookWaitExit`, not by a `hookWait` run. Pinned in pieces.
- Codex `HookInput` → `TestHookInput` rows `PostToolUse`, `StopContinued`, `OtherEvent` and `NotAnObject`. Pinned.
- Codex `HookOutput` block, dropped echo, `{}` with no mail → `TestHookOutput`, four rows. Pinned.
- Codex `RewakeOutput` returns nothing and 0 → the final assertion of `TestHookOutput`. Pinned.
- `WakeIdle` runs `queue --thread <id> --message <text>`, no session id is refused, a failing queue returns its output → `TestWakeIdle`. Pinned.
- `WakeIdle` falls back to `codexBinary(Config{})` when `LYM_CODEX_BINARY` is empty → none. `TestWakeIdle` always sets the variable. See F1.
- `WakeIdle` with a payload that is not JSON, and the `queueTimeout` expiry → none. The first is a trivial error branch and the second is hard to pin deterministically, so I raise no finding on either.
- `creelMailHookCommand` with no flag (prints `{}`, runs nothing) and with the flag standing (passes `hook-read` output through) → `TestCreelHookCommands` rows `MailHookNoFlag` and `MailHookFlagStands`. Pinned.
- `creelStopHookCommand` with no flag, mail handed out, and nothing handed out → `TestCreelHookCommands` rows `StopNoFlag`, `StopHandsOutMail` and `StopHandsOutNothing`. These rows also check that the turn end is recorded only when no mail was handed out. Pinned.
- `appendRecord` / `recordingCommand` split → `TestHookCommands_RecordAndStamp` and the `TestHookSettings` literals. Pinned.
- `hookSettings` with creel (Stop setting swapped for the merged command plus the async waiter, creel mail hook after the activity hook) → `TestHookSettings` row `Creel`. Pinned.
- `asyncHookEntry` / `creelWaitHookCommand` text → the literal waiter assertion in `TestHookSettings`. Pinned.
- `Prepare` marks `CreelHooks` and builds `creelDelivery` for an engine told a board → `TestPrepare_CreelHooksInAHub`. Pinned, with the weakness in F1.
- `Prepare` with no board adds no creel hooks and no `CreelHooks` mark → `TestPrepare_Launch`, whose `reflect.DeepEqual` on the launch uses `creelDelivery{}`. Pinned.
- Trust sentence in the Codex prompt file → the `prompt.md` assertions in `TestPrepare_Launch`, `TestPrepare_InstallsGuardsFromItsInputs` and `TestPrepare_AppendsStandingPassages`. Pinned for a non-empty deny notice.
- `promptText` with an empty deny notice (trust sentence alone at the head) → none seen. I found no direct `promptText` test, and I did not confirm that any Prepare test runs with an empty notice. See F2.
- `OpenLaunch` carries no creel hooks → the existing `open_test.go` assertions, unchanged by the diff. Pinned.
- `ToolBoundaryRenderer("codex")` is an `IdleWaker` → `TestToolBoundaryRenderer`. Pinned. The diff changes no production file in `shuttleengines`, so this row only pins that the existing registry entry carries the new capability.

**Test economy:** I found no duplicates and no assertions on static prose. `TestHookSettings` row `Creel` and `TestPrepare_CreelHooksInAHub` both touch the creel settings. The Prepare test pins the mark and the wiring, so I am not flagging the overlap.

**Findings**

F1
- Severity: LOW
- Class: design
- Location: `internal/shuttleengine/codexengine/prepare_test.go:299` and `internal/shuttleengine/codexengine/capabilities_test.go` (`TestWakeIdle`)
- Summary: nothing pins that the waiter's `LYM_CODEX_BINARY` comes from `codex.yaml`'s `binary`, or the `codexBinary(Config{})` fallback in `WakeIdle`.
- Evidence: `TestPrepare_CreelHooksInAHub` builds its expected settings with `bin: "codex"`, the default. A `Prepare` that hard-coded `"codex"` into the waiter would still pass. `TestWakeIdle` always sets `codexBinaryEnvKey`.
- Missing row: a configured non-default `Binary` in `TestPrepare_CreelHooksInAHub`.

F2
- Severity: LOW
- Class: design
- Location: `internal/shuttleengine/codexengine/codexengine.go:225`
- Summary: the changed `promptText` has no pinning row for an empty deny notice, where the trust sentence alone heads the file.
- Evidence: the grep finds no test calling `promptText`. The Prepare prompt assertions all concatenate `denyNotice(...) + " " + CreelTrustSentence`. I did not confirm that `installedGuards(nil, guardInputs{})` is non-empty, so the empty-notice branch may be uncovered.
- Missing row: an assertion on a launch that installs no guard rows.

**Tool calls**
1. `grep promptText(` and the `bin:` / `Binary` lines in `prepare_test.go`: to check whether the empty-notice prompt and a configured binary are covered, which the manifest did not show.
