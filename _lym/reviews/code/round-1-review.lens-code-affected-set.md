```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I found no finding.

**Per change, the set I found and whether it is covered:**
- **Trust sentence (`CreelTrustSentence`):** the only copy of the old wording is in `internal/shuttleengine/creelhooks.go:15`, and the diff rewrites it. The other hit, in `docs/research/codex/engine-trial/…`, is a dated trial record and not a rule text. Covered.
- **Parent directive and orch role:** I found no other stencil or contract repeating the old "an orch strand" trust wording. The directive and the orch role both gained the stamp lines. Covered.
- **New `creel` config module:** the modules registry (`configreg`) has the row. The wake-up and strictness tests (`daemonwakeups_test.go`, `configstrictness_test.go`) and the `docs/overview.md` module and package lists are updated. `PATTERN-config-layers` lists `creelconfig` as a degrading caller. Covered.
- **PATTERN declared change:**
  - `PATTERN.md` carries the declared `PATTERN-agent-addressing` text verbatim.
  - `PATTERN-agent-addressing.md` gains an "Across hubs" section.
  - `PATTERN-daemon-wakeups` lists the outbound worker.
  - `PATTERN-test-isolation` gains the `XDG_STATE_HOME` line.
  - The Slack rule is still in the background file (line 23). Covered.
- **Stamped message line, reminder and read header:** `MessageLine`, `composeReminder` and `writeMessagesText` carry the stamp. `messageFields` adds the `trust`, `sending_hub` and `sending_id` fields, and `from` stays bare.
  - `messageFields` also renders the `reply`, `returned` and `unanswered` fields of `post --wait-reply`, so the wait-reply JSON is covered through it.
  - The `lym creel log` entry fields are covered.
- **`reportengine` reply counts:** `ComputeReplies` handles the `inbound` and `outbound` events. Covered.
- **`standalonestate` directory split:** `UserStateDir` is added and `stateRootFor` is shared. Covered.
- **New daemon verbs and the `hubs` verb:** the `deliver` and `confirm` request fields, the `hubs` verb in the group help, the `post` and `forward` help, the `creelcli` and `creelengine` package docs, and `docs/overview.md` are updated.
- **Journal events:** I found no other event enumeration outside `creelengine` and `reportengine`. The `log` help text at `cli.go:749` is prose, and I did not check whether it lists the new events.

**Tool calls:**
1. I grepped the repo for stale copies of the old trust wording, module lists naming `reportconfig`, and `Outcome` and `messageFields` consumers in `creelcli`. My context lacked other callers of these.
2. I grepped the `log` help and event strings, the Slack rule in the `PATTERN-agent-addressing` background file, and `creelcli` doc mentions of sandbox. My context lacked the unchanged parts of these files.
