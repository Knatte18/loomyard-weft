```yaml
lens: repo-invariants
outcome: reported
```

Task: the repo-invariants lens over cards 1–12, checking test seams, doc text that a card falsifies, and card order.

**Result: every invariant holds; one LOW finding.**

**Checks shared by all cards**
- **Seam markers:** no `//lym:seam` marker exists in any test of slackengine, slackcli or hubgeom, so `seam-test-unlisted` cannot fire. No reflective seam test covers `slackengine.Geometry`, `RelayDeps` or `slackCLI`. The one `reflect.` user, `internal/hubgeom/geometry_integration_test.go:285`, compares env maps. Held.
- **New test files and tiers:** no card creates a `_test.go` file path. Every new test function is a handle in an existing test file, and each card states tier0 (card 5's beamcli test is tier1). This matches the files' current `tiergate.Require(t, 0)` calls (`internal/slackengine/relay_test.go:162`, `internal/slackcli/cli_test.go:362`). Held.

**Per card** (S = test seams, D = doc text, O = card order):
- **Card 1:**
  - S held.
  - D: it changes `API` to the app-level token and adds `OpenConnection`. That falsifies `internal/slackengine/doc.go:26` ("bot token as a bearer header") and the `PATTERN-agent-addressing` Slack rule ("reads the bot token only"). Card 12 edits doc.go and card 5 edits the pattern background, so held at the gate.
  - O: the `AuthTest` re-sign's callers (`slackChecks`, `login`) are on the card. Held.
- **Card 2:** S held; D: doc.go is covered by card 12's Edit; O: it uses `OpenConnection` and `NewAPI` from card 1. Held.
- **Card 3:** S, D and O held.
- **Card 4:**
  - S: `TestSlackGeometry` and `newGeometry` are listed. The slackcli harness literal uses keyed fields, so the new fields compile there. Held.
  - D: `internal/hubgeom/doc.go:23-26` is under Extends. Held as an invariant; whether Extends is the right type is not this lens's question.
  - O held.
- **Card 5:**
  - S held.
  - D: deleting `ReadToken`/`WriteToken` falsifies `internal/slackengine/doc.go:8`. Login's change falsifies `internal/slackcli/doc.go:5`. Card 12 Edits both files. The pattern background is edited. `PATTERN.md:43` is fenced unchanged, which satisfies `pattern-line-unprescribed`. Held.
  - O: every `ReadToken`/`WriteToken` caller from the grep (cli.go, cli_test rows inside `TestSlackCLI`, `configure`, files_test, inbound_test `newHarness`, relay_test row, beamcli integration test) is listed. Held.
- **Card 6:**
  - S held.
  - D: setup's change falsifies `internal/slackcli/doc.go:5` ("`setup` the hub's channel and operator"). Card 12 edits it. Held.
  - O: the `ConversationInfo` callers (setup, `slackChecks`, `TestAPI`/`documentedAnswers`, the fake's default answer) are listed. Held.
- **Card 7:**
  - S held.
  - D: deleting `PostMessage` falsifies `internal/slackengine/doc.go:29`. Card 12 edits it. Held.
  - O: `API` gets every method of the new `SlackClient` from card 1 with matching signatures. `fakeSlack` gains them, and `recordingSlack` goes. The `runOutbound` re-sign's caller, `RunRelay`, is listed. Held.
- **Card 8:**
  - S held.
  - D: `holder.go` adds an unbounded loop that `pattern/PATTERN-daemon-wakeups.md` (the Slack bullet near the end of "What counts") does not list until card 12 edits it. `unboundedWaitFiles` is extended on this card, and the constants are all ≥1s, so the scan passes. Held.
  - O: it uses cards 2, 4 and 5. Held.
- **Card 9:** S held; D: doc.go is covered by card 12; O: it uses card 3, card 4, card 5's `LiveHubs` and card 7's fakes. Held.
- **Card 10:**
  - S held.
  - D: removing the cursor falsifies `internal/slackengine/doc.go:11` and `:37`. Card 12 covers it. `internal/hubgeom/doc.go` names no cursor, and the card lists it under Uses. Held.
  - O: the `CursorPath` builders (`hubgeom.SlackGeometry`, `TestSlackGeometry`, `newGeometry`, slackcli `newHarness`) are all listed. Held.
- **Card 11:** S held; D: the `outbound.go` header is named in Intent; O: it uses card 9's keeper, and the re-sign callers `RunRelay` and the `NewOutbound` export are listed. Held.
- **Card 12:** S held; D: it edits both package doc.go files, `docs/overview.md`, `role-slack.md` and the `PATTERN-daemon-wakeups` background, and `relay.go` stays on `unboundedWaitFiles`; O: the callers of `runInbound` and `awaitOutboundParked` are listed. Held.

Outside the lens, noted only: `PATTERN-cwd-resolution`, `PATTERN-child-processes` and `PATTERN-config-layers` name `slackengine` or the relay. Their text stays true under the plan: no new process kind and no config key.

**Finding:**
- **Severity:** LOW
- **Class:** consistency
- **Location:** `_lym/plan/12-relay-push-loop.md`, the Docs paragraph on `internal/slackengine/doc.go`
- **Summary:** Card 12's doc.go prescription names only themes (push route, channel per orch, two tokens, manifest, stop archives nothing). It does not name the stale sentences that cards 1, 5, 7 and 10 leave in that file:
  - line 8, `WriteToken`/`ReadToken`, deleted by card 5;
  - line 11, the inbound cursor, removed by card 10;
  - line 23, the `RequireConfigured` "no token" wording;
  - line 26, "bot token as a bearer header", which card 1 makes untrue;
  - line 29, `PostMessage`, deleted by card 7;
  - line 37, the inbound poll;
  - lines 56–57, "inbound poll and the outbound read … five-second floor".

  `doc-go-unlisted` passes, but the doc can keep naming deleted symbols unless the card lists those sentences for rewrite.
- **Evidence:** `internal/slackengine/doc.go:8,11,23,26,29,37,56-57`; card 12's Docs paragraph.

Tool calls:
1. Grepped `//lym:seam` markers and Slack mentions in `pattern/`, `PATTERN.md` and docs: the manifest held no seam-marker or pattern-wide Slack search.
2. Grepped Slack wording in other packages' docs and the test tier calls: the manifest covered only the slack packages.
3. Read the Slack lines in the fabriccli, beamcli, orchcli and lymdirs docs: checked whether the plan falsifies them (it does not).
4. Grepped reflective tests and readers of `last_slack_ok`: checked for a reflective seam over the changed structs.
5. Read `hubgeom/geometry_integration_test.go`'s reflect use: confirmed it is no seam over `slackengine.Geometry`.
