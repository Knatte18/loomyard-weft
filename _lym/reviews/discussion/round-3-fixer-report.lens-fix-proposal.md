```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review's proposals that the fixer carried out. F2 and F3 were fixed as proposed, and F1 was fixed differently. Each finding below is a concrete case where a proposed change goes wrong.

1. **[MEDIUM] F3, D3: "When creel refuses that first read, the relay … ends, releasing its lock".**
   - **Summary:** ending the relay on any refused first read takes down the real relay on a passing refusal, and with presence `terminal` nothing starts it again.
   - **Case:** a prime anchor change ends and starts the relay while one live strand's pane process is briefly unknown. The record already treats this state as real (D4, D6). Creel refuses the relay's program claim, so the relay ends. With presence `terminal`, no dobby tick restarts it, because D3 restarts only while presence is `slack`.
   - **What breaks:** the D3 acceptance criterion "the relay is live after a prime anchor change with presence `terminal`, whether the change lands or fails" fails in that state. Orch replies to `[slack]` mail then wait in creel until the operator runs a start.
   - **Second case:** the re-exec path in D3 ("taking its lock again before its next poll") does not say whether the new image's first read counts as a "first read". If it does, a passing refusal at re-exec ends a relay that was running.
   - **Missing:** the change states no bound for this. It also does not say whether a creel daemon that is down at the first read counts as "refuses".

2. **[MEDIUM] F2, D6: the unknown-pane refusal that now applies to `stop`, `back` and `away` from the operator's terminal.**
   - **Summary:** a strand whose pane process stays unknown blocks the operator's own terminal from `lym slack stop` and `lym slack back`, and the stated way forward may never come.
   - **Evidence:** `internal/reedengine/lifecycle.go:76` says `PanePID` is "0 when the strand's pane is in no listing or is dead". So a strand that is still recorded live but whose pane is gone keeps the pid unknown until it ends. `internal/creelengine/identity.go:129` treats `&lt;= 0` as unknown.
   - **Case:** a live-recorded strand has lost its pane. From the terminal, `lym slack stop` and `lym slack back` both refuse with "waiting until its pane process is known or it ends", and neither may happen without a separate cleanup step that the record does not name.
   - **What breaks:** the Slack text `back` still works, since the relay handles it without creel. But `stop`, the relay's off-switch, has no path in that state. The relay also cannot forward to creel (D4), yet it keeps polling.
   - **Missing:** the proposal makes `stop` fail closed with no stated bound and no reachable way forward.

3. **[LOW] F1 (fixed differently), D2: "a setup file under the hub's `.lym/slack/` directory".**
   - **Summary:** the per-hub file sits in a directory the prime orch's pane can write directly, so it is no more protected than the config layer it replaced. The new Bound line covers this ("writes the files directly"), so the change holds.
   - **Remaining gap:** the D2 Rationale sentence "such a write would skip setup's checks" is just as true of a direct write to the setup file. The rationale only holds against `lym config`, not against direct writes. A reader may notice; nothing ships wrong.

Tool calls:
- `grep -rn PanePID internal`: my context did not show where `PanePID` is set or when it stays 0.
