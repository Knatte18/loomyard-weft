```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens, checking that each set-premised fix covers every member.

**Findings left unchecked:** none. F1, F2, F4, F6, F8 and F10 have set premises that I checked by search or against the files in context. F3, F5, F7 and F9 name closed lists, which I checked against the changed text.

**no findings**

- **F1:** `RoleKey` is the only link between runs for carry-over and crash-resume, and `roleKey` now returns empty for the agent. The other readers of a running record go by process or step, not role.
- **F2:** both launch sites an unawaited spec can reach (`launch` and `relaunchUnderAccount`) wrap the pane lines. The unwrapped `relaunch.go:61` is gated-only, and `StartSession` passes no gate.
- **F4 and F6:** both `roleFromFlag` callers are now tested. The only pause-refusing starts are `Open` and the agent verb.
- **F8:** no other help text restates the role rule. The button `ask:` strings reword it, but they are YAML that bans commas and are outside the finding.
- **F10:** both shuttle verb lists in `docs/overview.md` now match.

**Outside my lens, not verified:** with `EndsWithSession`, a provider that dies during startup now also removes its own strand. That could race `abandonStartup`'s removal and leave `startupStrandRemoved` false, which would skip the start retry. The correctness lens should check this.

**Tool calls (4 of 6):**
1. Searched the repo for the role-rule wording (F8), since my context held only the files the review named.
2. Searched for shuttle verb lists (F10), since only the two edited and cited lines were in context.
3. Searched for `AddStrand`, `RelaunchStrand` and `Prepare` sites (F2), to find any launch site not already read.
4. Searched for readers of run records, `RoleKey` and `PID` (F1 and F2), since consumers outside the files read were not in context.
