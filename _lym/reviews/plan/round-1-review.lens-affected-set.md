```yaml
lens: affected-set
outcome: reported
```

Affected-set lens: for each card's change, what reads or asserts it and which cards cover that set.

**Per change: the set found and the cards covering it**

| Change | Set found | Covered by | Verdict |
|---|---|---|---|
| New `creel` config module | `configreg.go`, `configreg_test.go` (both pinned chains), `configstrictness_test.go`, `PATTERN-config-layers.md`, `docs/overview.md` | Card 2 | Whole. `configreg/doc.go` names no module. |
| `setTempEnv` sets `XDG_STATE_HOME` | `TestSetEnv`, `PATTERN-test-isolation.md`. Standalone tests set the variable themselves. `HOME`-setting tests are in claude/codex engine, reed serverconfig, tmuxkit and battencli. No `_test.go` assigns a `[]string` literal to `.Env` | Card 1 | Whole, as far as searched. |
| `UserStateDir`, new `standalonestate` export | `TestDeriveCallerSet_CliwireOnly` scans only for `Derive` calls, so the `creelcli` caller passes | Cards 1, 4 | Whole. |
| `MessageLine` / `composeReminder` stamp | `deliver.go:408,437`, `hookread.go:123`, `hookwait.go:101`, echo at `verbs.go:627`, claim budget at `verbs.go:448`, `lineOverHookField`. `loomcli/decision.go:349,357` uses local `LineFits` only | Card 3 (all go through the one function) | Whole. |
| `Message` fields and renderings | `messageFields` (`cli.go:290`, feeding `reply`, `returned`, `unanswered` and the list), `writeMessagesText`, wire `Response.Message(s)` | Card 3 | Whole. `orchcli/creelseam.go:68` and `GoClient` read only program mailboxes, which card 6 and card 9's forward refusal keep cross-hub mail out of. `inbox` is a count, not a rendering. |
| New journal events and `LogEntry` fields | `logEntryFor`, `logEntryFields`, carry, prune and `moveOrigins`, `reportengine.ComputeReplies`, `orchcli` log folds (by id, local) | Cards 3, 9 | Code whole. Docs: see F1. |
| Notice names `log --since 1h` and `--from` | `parseSince` (`cli.go:769-773`) accepts durations | No change needed | Held. |
| Refusal for a name of another hub | Only `verbs_test.go:215` (`zz:orch`) uses such a name across the creelengine, creelcli and orchcli tests. It stays with a nil seam and no `Interrupt` | Cards 4, 6 | Whole. Card 6's "no test asserted" holds. |
| `Client.exchange` refusal type | `CallWatching`, callers of the `Error` text (text unchanged) | Card 8 | Whole. |
| `Options.Hubs`, `serveDaemon` | `Run` callers: `cli.go:962`, `cli_test.go` `startDaemon`, `server_test.go` `rig.options` | Cards 4, 9 | Whole. |
| `lym creel hubs` verb | `Command` Long, `creelcli/doc.go`, `docs/overview.md:323` and `docs/overview.md:512` | Card 5 | Partial: see F2. |
| `CreelTrustSentence` | Claude and Codex readers and tests compare against the constant. `claudeengine` panics on a line break, and the new text has none. No other copy of the wording | Card 10 | Whole. |
| Parent directive and orch role text | `parent-directive-parent.md`, `orch-role.md:57,309`. No Go test pins this prose. The `none` and `operator-ban` variants carry no trust line. `parentdirective/doc.go:8` never named orch strands, so it stays true | Card 10 | Whole. |
| `PATTERN.md` line | Background keeps the Slack rules. `agentmessaging_test.go` scans for SendMessage text | Card 10 | Whole. |
| `creelengine/doc.go` line 336 | Dropped by card 9, with sections added by cards 3 to 9 | Cards 3-9 | Whole. No spec or skill under `contracts/specs` or `plugins` names creel. |
| Worker waits in the sweep's duration range | `isSweepWait` helper | Cards 6, 8 (stated) | Held. |

**Findings**

F1
- severity: MEDIUM
- class: consistency
- location: `_lym/plan/09-replies-across-hubs.md`, `**Uses:**` line for `internal/reportengine/doc.go`, and the `ComputeReplies` paragraph
- summary: Card 9 makes `ComputeReplies` count `inbound` as `posted` and count `outbound` entries with `Answers` as `Replied`, but it lists `reportengine/doc.go` only under `Uses`. The package doc's counting rule becomes false, and the card covers the function but not this reader of its behaviour. The `RoleReplies` field comments ("Expected counts the posts…", "Replied counts the replies the role sent") are not listed either.
- evidence: `internal/reportengine/doc.go:206` ("Expected, the `posted` entries…; Replied, the `posted` reply entries…"); `internal/reportengine/replies.go:15-18,27`
- fix: Move `internal/reportengine/doc.go` from `**Uses:**` to `**Edit:**` on card 9 and have the Intent state the new counts.

F2
- severity: LOW
- class: consistency
- location: `_lym/plan/05-hubs-verb.md`, the paragraph naming `docs/overview.md`'s `internal/creelcli/` line
- summary: Card 5 updates the verb list on the package-tree line but misses the second verb list in the same file, the **creel** module bullet. Card 10's edit to that bullet covers only the daemon and the trust sentence, so no card adds `hubs` there.
- evidence: `docs/overview.md:323` (tree line) and `docs/overview.md:512` (`lym creel post`, `inject`, `interrupt`, `forward`, `read`, `inbox`, `log`, `hold` and `release`)
- fix: Card 5's sentence names both lines. The target `docs/overview.md` is already on the card.

One lead is unverified and is not a finding: `cmd/lym/sandbox_coverage_test.go:46` has a `Key: "hibernate"` entry, which may pin per-module coverage. My tool budget ended before I could read it, so I do not know whether the `hubs` verb or the `creel` module forces an entry there.

Tool calls:
- Grep of trust wording, config module lists, XDG and HOME tests, message-line consumers and foreign-name tests: the manifest held the definitions, not every reader.
- Grep of `.From` renderers, `Withdraw`, log-event readers, `protocol.go` and verb tables: the manifest lacked the wire types and the readers outside creelengine.
- Grep of docs, specs, parent stencil variants and PATTERN pins: the manifest read only `parent-directive-parent.md` and parts of `orch-role.md`.
- Read of `callerset_enforcement_test.go`, the `orchcli/creelseam.go` span, orchengine consumers and HOME-setting tests: needed to judge card 1's `UserStateDir` caller and card 3's "existing fields only" claim.
- Read of `parentdirective/doc.go`, the `none` and `operator-ban` stencils, prose pins, `contracts/specs` and `plugins`, and overview trust lines: these are readers of card 10's text outside the manifest.
- Grep of `configreg/doc.go`, `"hibernate"` module lists, `parseSince`, scratch-env tests, `reportengine/doc.go` and `messageFields` callers: these are readers of cards 2, 3 and 9 that the manifest lacked.
