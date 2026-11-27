```yaml
lens: fix-affected-set
outcome: reported
```

Task: lens fix-affected-set, checking that each set-shaped fix covers every member.

**Unchecked by a rerun search:** F1, F5, F6, F7, F9, F10, F11, F12, F13, F14, F16, F18. For F4, F9 and F12–F14 I judged only from context. The search for F4's doc references and F9's `webster reset` descriptions was refused by a hook that matched the literal text in my grep.

**Searched:** F2/F3 (ledger-entry consumers), F17, F8, F15.

**Findings**

1. **MEDIUM**: `internal/runpusher/parent.go` `payOwedSteps` and `HandOvers`. An owed reset or hand-over with no landing left and no refused-push record is never retried, so part of F2's premise stands.
   - The owed step is persisted before it is made, but `targetAhead` (`internal/runpusher/targetreads.go:23-44`) reads neither `HandOvers` nor a carried `Dropped` commit as ahead.
   - `restartHubPushers` starts through `StartIfAhead`, so no pusher spawns. It calls `SettleRefused` only for a refused target.
   - Concrete case: `settleOnOrigin` moves the only landing into `HandOvers`, then `payOwedSteps`' `p.RunOwner` fails and returns `owed(err)`. `pushUntilLevel` loops, `Ahead` is false, `settleLevelEntry` keeps the entry and the pusher exits with no record.
   - A pusher killed between the entry write and `ResetToOrigin` or `HandOver` ends the same way. Nothing runs the step until another landing arrives.
   - The new doc says "made again on every pass until they succeed", and no pass happens.
   - The `AFailedHandOverIsRecordedAndMadeAgain` row calls `SettleRefused` directly, so it does not show this.

2. **LOW**: `pushledger.Entry.HandOvers`, a member of the entry's consumer set that was missed. `internal/dobbyengine/pushsummary.go:81` lists a non-owed entry as held only for `HeldRecords != "" || len(Dropped) > 0`. An entry holding only an owed hand-over (the settled-on-origin path) is in neither list, so the summary treats it as nothing.

3. **LOW**: F5/F14 landing way forward, set of beam-parent push paths. Only the pusher's `PushTip` sets `Landings` (`internal/loomcli/pusher.go`).
   - The way forward a beam record names is `lym fabric push`, which builds `PairPush{LockWait, Here}` with no `Landings` (`internal/fabriccli/weft_verbs.go`).
   - If that hand push is refused again, as protected or diverged, it replaces the record. The protected case then reads "run `lym fabric push` to retry the push" again.

F8, F15 and F17: no missed member that matters. The remaining "detached push" comments still describe a detached process.

**Tool calls**
- grep of `.Dropped`, `.HeldRecords` and `.Landings` readers: my context did not hold the full consumer set.
- grep of stale push wording (F17): my context held only the sites the fixer edited.
- grep of `PushDeferred` and the sentinels (F8/F15): to confirm no member remained after the edits.
- grep for removed methods and `webster reset` text (F4/F9): refused by the hook, no output.
- `sed -n` of `pushsummary.go` and `landingstands.go`: neither was in my context.
