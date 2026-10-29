```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision end to end on one case.

**Decisions walked**
- **D1, met.** Case: a Stop with `stop_hook_active` false and one mail.
  1. `ClaimAtBoundary` sets the count to 0, claims the mail and raises the count to 1.
  2. `hook-read` prints `additionalContext` and exits 0. Through `&& exit 0` in `creelStopHookCommand`, no turn end is recorded.
  3. In the Claude Code Stop runner, `Ugt` is false outside bare or simple mode, so the context goes into `He`. That makes `blockingErrors` non-empty, so the turn continues with `stopHookActive` and the block count plus one.
  4. At the next Stop, `stop_hook_active` is true, so the count is not reset. The claim comes back empty, `hook-read` exits 1, and the record step writes one Stop payload.
  5. The operator sees `Stop hook feedback: <lines>` from `hookAdditionalContext`, drawn in gold rather than red, so no error label.
- **D2, met.** Case: one mail at a PostToolUse boundary. The claim hands it out and sets the count to 0. The output has `additionalContext` plus `systemMessage` and no block. The turn goes on unchanged.
- **D3, model side met; operator side not walked.** Case: an idle marked waiter, one claimable mail, `NoTurn`.
  1. `wakeHookStrand` calls `ringHook` with the ids, and the claim runs in the locked step.
  2. The ring's outcome carries the lines. `hook-wait` writes them to stderr, acks, gets the confirmation and exits 2.
  3. `_tt` reads stderr after the exit, because the code is 2. It builds `${rewakeMessage} ${stderr}`, passes it through the escape and the marker replacement (`bWe`, which does not truncate), and enqueues it with `Bin` as a task-notification with priority `next`.
  4. Failure walk: if the ack fails, `hook-wait` exits 0 and Claude Code does nothing. Release makes the claimable ids equal `mark.ids` again, so `sameSet` stops a re-ring, `waiting` is false, and the bell comes after `ringAnswerWindow`. This matches the acceptance criterion.
  5. Budget walk: the leftover set differs from `mark.ids`, and the woken turn marks the ring answered. The leftovers stay flagged for the next tool call or Stop. Held.
- **D4, model side met; operator side not stated.** Case: the rendered waiter entry with both keys. The model text is `[creel] Mail that arrived while you were idle: <lines>`. The operator summary is `rewakeSummary`, because the plugin-only override `We` is unset for a settings hook.
- **D5, met.** Case: a ring claim that hands out mail and is then released. The outcome is known only when `handOut` or the ring's ack wait finishes, so one line per attempt fits at that point.

**Findings**

1. **LOW · design · D3 and D4 (ring wake, operator copy)**
   The record never says what the operator reads when the ring hands mail over, and no acceptance criterion checks it. Without that, the Goal's "the operator once" is not shown to hold for the ring path.
   - The steps I traced end in `Bin({summary, body})`. It enqueues `Di({summary, trailing: body})` as a `task-notification`.
   - D4 states only the summary label `creel mail`. The D3 and D4 acceptance criteria check the model side and the settings keys, not whether the operator sees the mail body once, or at all.
   - Evidence: the 2.1.312 binary, `function Bin({summary:e,body:n,...})` (the dd dump above), and `_tt`'s `Bin({summary:bWe(rt),body:bWe(rx(...)),priority:"next",stopHookActive:!0,...})`.
   - Fix: state in D4 what the operator's terminal shows for a ring wake, as source evidence. Add a ring-path operator check to the D3 real-session acceptance criterion.

No other step handed a later step different data than the record says.

**Tool calls**
- Helpers `rx`/`kjt`/`wjt`/`bWe` in the binary: needed to check whether the 8 KiB rewake body is truncated or misread as a broken-script error on its way to the model. The manifest had only their call site.
- `Bin` prefix, plus `creelStopHookCommand` and `creelWaitHookCommand` in `internal/shuttleengine/claudeengine/settings.go`: needed the `&& exit 0` turn-end rule of the merged Stop command and the waiter command. Neither was in context.
- `Bin` body, attempted with grep: the output came back truncated at a newline.
- `Bin` body via dd: needed what the rewake notification carries (summary and trailing body) to walk the operator side of D3 and D4.
