```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. For each change, the other places it touches, and whether the diff covers them.

**Per change, the set found and whether it is covered**

- **`shuttleengine.Engine.KeepWarmCaps` (new method).** The implementers are the claude engine, `internal/shuttleengine/fakes_test.go` and `internal/testkit/shuttlefake`, and all three are updated. Covered.
- **`cardverify.Runner.RunLine` and `RunCards` take a context.** The callers are `websterengine/cardverify.go`, `gatecli/cardverify.go` and `cardverify/runner_test.go`, and all are updated. Covered.
- **`slotWait` and `answerSlotBusy` removed.** A grep for `slotWait`, `answerSlotBusy` and `no gate slot is free` found no leftover in non-test code except the front's own reworded message. `RunGate` and the `newCommand` callers are updated. Covered.
- **`GateCall` split into `AgentGateCall` (D7).**
  - The agent-rendered sites use `AgentGateCall`: the card and batch gates, both `webster-fix` sites (`verifyfixer.go:103`, `runlevel.go:699`).
  - The Go-run sites keep `GateCall`: `verifygate.go:223`, `loomcli/roundgate.go:18`, `webstercli/verify.go:63`.
  - The Verify-line texts keep `--wait`, as D7 requires. These are `loom-plan-spec`, `loom-rubric-plan-review`, `loom-block-plan-cards`, `plancontext.go` and `verifymodule.go`.
  - Covered.
- **Agent texts for the 75 answer (D9).**
  - `shellguard/rows.go` (both gate-rule strings), the implementer, darn-write and drift-verify stencils, the `await` help and the `DefaultAwaitCap` doc are all updated.
  - A grep for exit-75 and 300000 wording in `contracts` and `internal` found no other agent-facing text still reading "no slot free yet" or a 300000 ms timeout.
  - The only hits left are `docs/research/*` benchmark snapshots, which record past runs and are out of scope.
  - The llm driver stencil is not in the diff stat (D8). Covered.
- **New config keys `keep_warm_cap`, `keep_warm_cap_roles` and `keep_warm_max_reruns`.**
  - They are in the templates, in the specs as `CategoryNone`, and `keep_warm_cap_roles` is in `ConfigOpenMaps`.
  - `LoadConfig` validates `keep_warm_max_reruns`, and `shuttleengines.CheckLoad` calls `ValidateKeepWarm`, so `lym config claude` is covered.
  - `configsync.formerTemplateValues` lists only replaced units, so the new keys need no row.
  - Covered, apart from the docs gap below.
- **`runproc.RoleGate`.** Added to `record.go` and to the `doc.go` role list. Covered.
- **Wake-rule registry for new unbounded waits.**
  - `cmd/lym/daemonwakeups_test.go` lists `detachrun.go` and `detachfront.go`.
  - `pattern/PATTERN-daemon-wakeups.md` has both entries. Covered.
- **Declared PATTERN change.** `PATTERN.md` carries `PATTERN-keep-warm-cap` with exactly the declared text, placed under Processes and waits. No background link is declared. The `PATTERN-test-speed.md` edit matches the `GateCall`/`AgentGateCall` split. Covered.
- **`RunRecord.CallID` and `Abandoned`.** The `gateslot/doc.go` and `report.go` help text and the tolerant-read tests are updated. Covered.
- **Commit messages.** Every message is a bare `N: card-slug` plus the Co-Authored-By trailer. There is no `Pattern-Departure:` trailer, so none needs judging, and nothing in the diff departs from a PATTERN entry.

**Findings**

1. **LOW, consistency.** `docs/overview.md:419`, the shuttle paragraph.
   - It describes the per-role prompt-cache TTL from `claude.yaml` ("The engine sets the prompt-cache TTL per role… from `prompt_cache_ttl` and `prompt_cache_ttl_roles`").
   - It does not name the new `keep_warm_cap` and `keep_warm_cap_roles`, though D4 puts them beside those keys and the same paragraph now says `await`'s cap defaults to the role's keep-warm cap.
   - The diff touched this paragraph for `await` but left the config description incomplete.
   - Fix: add one sentence naming `keep_warm_cap` and `keep_warm_cap_roles` and the 250 s template cap.

Nothing else was left wrong in the affected set.

**Tool calls**
1. Bash grep for stale gate text, `PromptCacheLifetime` and `KeepWarmCaps` implementers, and `RunLine`/`RunCards` callers; my context had none of these call-site lists.
2. Bash grep for `GateCall`/`AgentGateCall` callers, exit-75 wording, and `keep_warm` keys in templates and docs; same reason.
3. Bash `sed` of `internal/configsync/configsync.go` plus the diff stat for `configsync`, `configreg` and `docs/overview.md`, to check whether the new keys need a `formerTemplateValues` row; the diff showed neither file.
