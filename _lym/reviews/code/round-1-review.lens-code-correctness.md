```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each changed function against concrete inputs.

**Per-function list (case run → verdict)**

- `slackLineExecutable`, `composeSlackToggleCommand`, `composeSlackStateCommand` (`panebin.go`):
  - Test binary, stub returning the same path as `os.Executable`: refused, no Slack line pinned. Correct.
  - Stub returning another path: pinned, harmless. Correct.
  - `os.Executable` failing under test: refused. Correct.
  - Production, `testing.Testing()` false: path passes through. Correct.
  - The toggle and state lines are dropped together, and `switchBindingCommandsLocked` still returns the other four lines. Correct.
  - Other `executablePath` lines (follow, button, switch) run only on a click, so they cannot loop at a redraw. `lym slack hook` is resolved through PATH, not `executablePath`. No loop found.
- `TestMain` (`testmain_test.go`): `reedengine.test slack bar` has `NArg=2`, so it exits 2 before `tmuxkit.Main`. A normal `-test.*` run has `NArg=0`. Correct.
- `bindingArgvs` / `slackToggleCondition`: `slackClick` false leaves the branch unbound, and the user ranges `slack` and `away` match the formats. Correct.
- `slackToggleFormat` / `awayToggleFormat`: empty job output is unlit, and `slack-off away-on` does not match `*slack-on*`. Correct.
- `barLine`: `machine()` error gives both off, and an error from `Connected` gives off. Correct.
- `Connected` against `holder.takeSocketLock`: the holder blocks in `AcquireWriteLock`, so a probe taking and releasing the free lock does not make it give up. Correct.
- `toggle slack`:
  - Connected goes to `switchOff`. Not connected goes to `start`, which clears the switch, then `ensure`. Correct.
  - `switchOff` writes the switch before ending relays, de-duplicates the current hub, and keeps going past a failing hub. Correct.
  - A relay of a hub that `liveHubs` does not list stays up, and the toggle still reads connected. Edge, not a defect.
- `toggle away` (via `back()` / `away()`): `away()` clears the switch, then starts the relay. Correct.
- `WriteSwitchedOff` / `ReadSwitchedOff` / `EnsureRelay`: a missing file reads as on, and an unchanged write is skipped. A corrupt switch file makes every start return an error, which is acceptable fail-loud. Correct.
- `WritePresence`: the pre-read short-circuits an unchanged value, ignores a read error, and falls through to `UpdateJSON`. Correct.
- `awayOnOperatorMessage` (`inbound.go`): runs after the `away`/`back`/`status` switch and before `forward`. A failed write is logged and holds nothing. It is equivalent to "only when terminal", because `WritePresence` skips a repeat. Correct.
- `noticePresenceChange`, `followPresence`, `diskCheck` (`relay.go`):
  - `noticed=""` at start takes the value silently. A failed beams read gives a hub-only notice and does not record `w.presence`, so it retries. Correct.
  - A change made while the relay is down or re-executing is never noticed, because `startPresence` is re-read. Documented in the summary, not a defect.
- `promptHook` (`hook.go`):
  - Empty prompt, or a `[creel]` or `[lym]` prefix: no switch. Correct.
  - Presence not slack: returns before any creel read. Correct.
  - An inject matches by `Subject`, which for injects is the full text (`fold.go:663`), and both sides are trimmed. Correct.
  - Failure path: a failed creel `Log` returns an error and leaves presence slack. Fail-safe, but see F1.
  - The `[lym]` lines that tender, shuttle and pause actually type all carry the prefix. Correct.
- `replyHook`, `fitReply`, `orchClient`:
  - Empty reply: nothing sent. Presence not slack: nothing sent. A non-orch strand is refused with a way forward. Correct.
  - `fitReply` at 7000 bytes, 7001 bytes, and a multibyte rune straddling the cut: always at most 7000 bytes, cut at a rune start. Correct.
- `HookInput` (`capabilities.go`): Stop gives a trimmed `Reply`. Prompt goes through `typedInstruction`, which returns empty for a task notification and a slash command. `Subagent` is checked first in `hook()`. Correct.
- `buildSettings` / `recordHeldStop` / `turnHookStep`: the hook input is held in `$in`, the recording's exit status is kept, and the turn hook runs in a subshell with output discarded. In the merged creel Stop command it runs only when no mail was handed out. Correct.
- `outboundRoute` / `relaySender` (`outbound.go`): the sender is `<machine>::<name>` for orch, program and relay. Correct.
- `SlackMachine` / `SlackGeometry`: the shared paths flow into `Geometry`. Correct.

**Findings**

- **F1 — LOW, design.** The turn hook's output and errors are discarded (`claudeengine/settings.go:155`, `>/dev/null 2>&1`), and `hook.go` writes no relay log or logger line on failure.
  - A creel `Post` failure in `replyHook` loses the reply with no trace.
  - A creel `Log` failure in `promptHook` leaves presence slack with no trace. Evidence: `hook.go:140-143`, `hook.go:177-179`.
  - Fix: log the failure with `logger.Warn` in `promptHook` and `replyHook` before returning the error.
- **F2 — LOW, consistency.** `barLine` and the lit state of `slack` follow lock-held. A holder whose dial is refused forever (bad token) keeps the lock and lights `slack` green. `switch.go:50-65` documents "reconnect reads as connected" but not a permanently failing dial. It is covered by the summary's design note, so I report it only as a possible mislead, not a defect.

I found no BLOCKING or MEDIUM defects in my lens.

**Tool calls (3 batched, then 1 and 1)**
1. `git diff` of `slackcli/cli.go` and `wiring.go`: my context lacked `start`/`away`/`back`/`setPresence` and `worktreeFreeVerbs`.
2. Grep of the `slackengine` lock call sites: needed to see how the holder takes `SocketLockPath` against `Connected`.
3. Grep of `typedInstruction` and `Subject` in `fold.go`: needed the prompt and inject-match semantics.
4. Read `slackengine/holder.go` 95-195: needed the holder's blocking `AcquireWriteLock`.
5. Grep for `[lym]` constants and pane-typing call sites: needed to check which lym-typed lines carry the prefix.
