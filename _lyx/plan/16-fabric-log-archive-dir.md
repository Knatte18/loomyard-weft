# Card 16 — The hub log archive root

**Create:**
- `plan:internal/fabricengine#LogArchiveDir` -> `func LogArchiveDir(hub string) string`

**Edit:**
- `internal/fabricengine_test#TestHubScratchAndLogsDir`

**Uses:**
- `internal/fabricengine#HubScratchDir`

**Intent:**
D14's archive root: one accessor in `fabricengine`, beside `HubLogsDir` in `junctionnames.go`, returning `filepath.Join(HubScratchDir(hub), "log-archive")`, which is `<hub>/_board/.lyx/log-archive`.
The archive is never tracked, so it lives under `.lyx` (`PATTERN-durable-vs-ephemeral-state`), and it sits under `_board`, reachable from the hub only (`PATTERN-hub-containment`); deriving it from `HubScratchDir` keeps `.lyx` spelled only by `lyxdirs` (`PATTERN-lyxdirs-single-declarer`).
Its doc comment says the pair teardown's copy and its prune both resolve the root through it.

`TestHubScratchAndLogsDir` additionally asserts `LogArchiveDir(hub)` equals `filepath.Join(HubScratchDir(hub), "log-archive")`, and its doc comment and `//testtiming:keep` reason name the archive root beside the logs directory.

**ImpactSummary:** one new path accessor in `internal/fabricengine` with no caller until the next card; one existing test gains an assertion.

**Commit:** 16: fabricengine: name the hub log archive root
