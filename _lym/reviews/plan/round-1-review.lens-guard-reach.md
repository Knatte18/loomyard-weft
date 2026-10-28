```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard, refusal, branch and test row a card adds, I checked whether a real entry point can reach it and whether the guarded site recurs on a path no card guards.

**Per guard, reachable or not**

- **Card 4, `validateLine` over-budget and empty-body refusals in `Store.post`:** reachable. A `post` with a long body gets there; nothing upstream blocks it, since `maxRequestBytes` is 1 MiB (protocol.go). The guarded site recurs unguarded on program notices (finding 1).
- **Card 4, `Store.Reply`:** a strand reply over the budget is refused, and a reply to EXTERNAL keeps 64 KiB. Both are reachable, through rows of `TestStore_Mail`.
- **Card 4, the claim skipping a line past `hookFieldCap`:** reachable. Production reaches it only through old-binary journals, and the test reaches it through `PostUnchecked`.
- **Card 4, no flag for a strand whose only mail is over the cap:** reachable.
- **Card 5, `decision add` refusing an entry whose notice does not fit:** reachable. A 9,000-byte decision passes every earlier check; `decisionEntries` has no size check (decision.go:127-199).
- **Card 5, one notice per entry in `notifyLiveRuns`:** reachable.
- **Card 6, the daemon-start cap, the reset on `StopHookActive` false and the seventh Stop claiming nothing:** all reachable.
- **Card 7, `NoTurn` and `TurnEndedAt` under the hook reading:** reachable. The pane-probe branch for an engine with `InterruptGuard` has no test that reaches it (finding 2).
- **Card 8, `PaneMail` true:** reachable only in tests. No production strand can reach it (finding 3).
- **Card 9, the probe typing notices and reminders only for `PaneMail`:** reachable in tests; production is as for card 8.
- **Card 10:** reachable in every part:
  - the `HookWait` refusal of a non-strand sender (card 13's EXTERNAL row)
  - the ring conditions and the hold from a ring until a later turn end
  - superseding an older waiter
  - reconnecting after a daemon restart (`awaitOutcome` returns on `d.done`, so the client sees `connLostError`)
  - nothing rung under a pause
- **Card 11, the grace rule, the 30-second window, `ErrNotLanded` recorded, `ErrBusy` retried, and a ring answered by a claim or by a running-turn reading:** all reachable.
- **Card 12, `HookInput` failing on non-JSON and an unrelated event mapping to empty:** reachable in tests.
- **Card 13:**
  - Stop with nothing claimed exits 1: reachable.
  - A subagent Stop exits 1: a guard the Constraints justify.
  - Refusing any other event: a fail-loud wiring guard, so exempt.
  - A block written then a failed acknowledgement: reachable only through `TestHookReadExit`, as the card says.
  - The `hook-wait` refusals (unknown engine, non-JSON input, EXTERNAL caller) and the superseded wait: reachable.
- **Card 14:**
  - The merged Stop command's branches are all reachable through the stub `lym`.
  - The command reads exit 0 from `hook-read` as "a block was written". No path exits 0 without writing: `clihelp.Abort` with code 0 never sets the abort flag (exec.go:99-120).
- **Card 15:**
  - The refusals before any key are all reachable: an invalid body, an unknown receiver, `checkMayAct`, the receiver without the mark, another sender's hold, a pause lifted before the key.
  - `--interrupt --reply` is reachable.
  - "Store fails after the key": the planned test row does not reach it (finding 4).
- **Card 16, the `init` check on the sentence:** a fail-loud wiring guard, so exempt.
- **Card 18, the interrupt-and-bell scenario:** whether it reaches the bell depends on undocumented waiter survival (finding 5).

**Findings**

1. **MEDIUM, design — card 4 (also bears on card 5): the line cap now refuses program notices no card bounds.**
   - `validateLine` applies to every post. Program notices are posted through `orchcli.NotifyParent` and `NotifyOrchAt` by burler, webster, batten, pawl and loom halt notices, and the orch watcher posts through `w.creel.Post`.
   - Like LOOM's decision notice, these programs cannot take the "write it to a file" way forward. A refused notice is lost: notify.go says "A refusal or failure comes back as an error carrying the full line, and the notice is not retried".
   - Only card 5 guards a program notice. No card shows the others stay under 8 KiB or handles their refusal.
   - Evidence: internal/orchcli/notify.go:60-75; internal/loomcli/wiring.go:577-579; internal/burlercli/wiring.go:152-155; internal/orchengine/watcher.go:612.

2. **MEDIUM, design — card 7: no test reaches the pane-probe branch for an engine with `InterruptGuard`.**
   - Claude implements `InterruptGuard`, so this branch is the production path for a marked Claude strand whose hook reading is unknown. In that branch a draft with no running turn reads `NoTurn` true.
   - The card's pane-probe rows all use `readinessEngine`, which by the card's own text implements no `InterruptGuard`.
   - `fakeEngine` implements it (`RunningTurn` is "RUNNING" in the capture) and could carry such a row.
   - Evidence: internal/shuttleengine/claudeengine/session.go:26,147; internal/shuttleengine/fakes_test.go:418-422; card 7's Intent.

3. **MEDIUM, decision — card 8 (and card 9): the `PaneMail` branch cannot occur in production, and the case the added decision names falls to no delivery.**
   - `PaneMail` needs a non-empty provider with no renderer. `shuttleengines` registers only `claude`, and only shuttle sets a provider. The card admits this.
   - Today a Codex session can exist only as a hand-added strand. It has no provider, so it gets neither path; today it gets the pane path.
   - The added decision says a strand whose engine has no hook delivery, "such as a Codex session, keeps today's pane path". The guard it requires is reachable only by a state that cannot occur, while the real Codex case loses delivery. This needs a parent ruling or a different test for the engine.
   - Evidence: internal/creelcli/delivery.go:264-311; internal/creelcli/smoke_helpers_test.go:87-102 (`reed add` sets no provider); decision-record.md:186.

4. **MEDIUM, consistency — card 15: the planned row for a store that fails after the key does not reach that branch.**
   - The row closes the store's journal inside the interrupt function. `interruptTarget` then appends `eventInterrupted` itself, after the key, through `return true, s.do(... append ...)`. It returns sent=true with the journal error before `post` reaches its store step.
   - The card treats `interruptTarget`'s errors as refusals "with nothing stored". So the row lands on an unplanned after-key failure, not the "interrupted, but the post was not stored" message.
   - The card should name that `interruptTarget` can return sent=true with an error and map it to the after-key message, or test the store step with a fault after `interruptTarget` returns.
   - Evidence: internal/creelengine/verbs.go:537-583.

5. **LOW, consistency — card 18: the interrupt-and-bell scenario depends on undocumented behaviour.**
   - It asserts the bell is typed exactly once after `post --interrupt`.
   - If a waiter started by an earlier `Stop` of the session is still live, D7 says it gets rung instead, and no bell is typed. D5 calls Claude Code's keeping of an older waiter undocumented.
   - The card does not make the interrupted turn the session's first turn, the one case with no earlier `Stop` and so no waiter.
   - Evidence: decision-record.md:115,141; card 18's Intent.

**Tool calls**

1. grep for `NewProgramClient`/`NewStrandClient` callers and `clihelp.Abort`/`ShouldAbort`: needed program posters beyond creel and whether an abort can exit 0.
2. orchcli/cli.go, tendercli/deps.go, orchcli/notify.go and the `.Post(` callers: needed what program notices post and how a refusal is handled.
3. `NotifyParent`/`NotifyOrchAt` callers and orchengine/watcher.go:600-616: needed which programs post unbounded notice lines.
4. loomcli/wiring.go:565-582, burlercli/wiring.go:140-156 and battencli notices: needed what goes into a notice line.
5. claudeengine `InterruptGuard` and `fakeEngine.RunningTurn`: needed whether the card 7 pane branch is Claude's production path and whether a fake could reach it.
