```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if the behaviour broke.

**Per behaviour, the test that pins it**

1. A fork spawned by a subagent leaves out the tool calls it inherited, matched against its spawner's transcript through `meta.ParentAgentID`. Pinned by `TestAuditForks_SubagentForkCountsOnlyItsOwnCalls`: on the old code `agent-last` reports 3 Agent calls and 1 Bash command, against the expected 1 and 0 (`audit_nested_test.go:359-361`).
2. A fork spawned by the session leaves out every tool call whose id is in the session's own transcript (`inherited := parent.toolUseIDs`, `audit.go:90`). **None.** No fork fixture replays a parent id other than the spawning call, and that call is already left out by `toolUseId`. The write-events test was changed so its ids do not collide with the parent's (`audit_test.go:453-466`).
3. `ForkReport.SpawnedBySubagent` is set from `parentAgentId`. Pinned by `TestAuditForks_SubagentForkCountsOnlyItsOwnCalls`, where `spawnedBySubagent` is true and false in different rows (`audit_nested_test.go:382`).
4. A spawning subagent's own calls are not left out (it is checked against the session's ids). Pinned by the `agent-gp.jsonl` row, which expects 2 Agent calls and 1 Bash command (`audit_nested_test.go:359`).
5. A missing spawning-subagent transcript fails the audit with an error naming the file. Pinned by `TestAuditForks_MissingSpawningSubagentTranscriptErrors` (`audit_nested_test.go:394-405`).
6. A spawner transcript is read once per audit (the cache). Not observable from outside, so no test is needed.
7. Chains deeper than two, a fork of a fork of a subagent, resolve through the intermediate transcript. This is the same code path as item 1, so it is covered by that test; it has no row of its own.
8. `auditFixRound` neither checks nor counts a subagent with `SpawnedBySubagent` set. Pinned by the `cluster_test.go` row "a fork a fresh subagent spawned is neither…" (`cluster_test.go:161-174`). Without the skip, the nested fork's write and `git commit` would break the round; `wantForks: 2` also pins the count.
9. The `auditParentTranscript` refactor into a struct. Pinned by the existing parent-facts and audit tests (`audit_test.go:204-245`, `audit_parentfacts_test.go`), whose assertions are unchanged.
10. The changed signature of `auditForkTranscript`. The spawn-boundary and streaming tests only had their call sites updated; their assertions are unchanged, so nothing was loosened.

**Findings**

- **MEDIUM | design | `internal/shuttleengine/claudeengine/audit.go:90`:** Leaving out a session-spawned fork's inherited calls by the session's own transcript ids is new behaviour with no test. Replacing `parent.toolUseIDs` with nil passes every test. The fixtures prove this: no `testdata/fork-*.jsonl` shares a tool_use id with `parent-spawns.jsonl` apart from the spawning call. The `audit_test.go:453-466` fixture was rewritten to avoid the collision instead of asserting it. The missing row is a session-spawned fork whose transcript replays a parent Bash or Agent call that `toolUseId` does not name, and whose report must leave that call out.
- **LOW | design | `internal/shuttleengine/claudeengine/audit_nested_test.go:55,61`:** The fixture's meta files use the key `parentAgentId` and the file-name layout `agent-&lt;parentAgentId&gt;.jsonl`, but no recorded Claude Code `.meta.json` backs either. Nothing in `testdata/` or `testdata/corpus` carries `parentAgentId` or `isFork`, and the sample of local session metas I checked had none. If Claude Code writes a different key, or an id the `agent-` file name does not use, every row still passes while production keeps the original bug (silently when the key differs; if only the id spelling differs, the spawner-transcript read fails and errors the audit). The board entry's table names a parent per transcript but not the meta key.

Tool calls:
- grep of fixture tool_use ids and the `copyFixture` call sites in the audit tests: my context held no fixture contents, which I needed to check whether any fork replays parent ids.
- listing of `testdata/corpus` and a search of fixtures and local session metas for `parentAgentId`: to check whether a recording backs the fixture's meta layout, which my context did not show.
