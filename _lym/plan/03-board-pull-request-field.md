# Card 3 — board-pull-request-field

**Edit:**
- `internal/boardengine#Task`
- `internal/boardengine#normalizeTask`
- `internal/boardengine#replayFields`
- `internal/boardengine#UpsertPayloadKeys`
- `internal/boardengine#scopeChanged`
- `internal/boardengine_test#TestNewTask`
- `internal/boardengine_test#TestRunLock`
- `internal/boardcli#storeVerbs`
- `internal/boardcli_test#TestHelpSchema_LeafCommands`

**Extends:**
- `internal/boardengine/doc.go`

**Uses:**
- `internal/boardcli/doc.go`

**Intent:**
D2: a board entry carries a per-entry pull-request override.
`Task` gains `PullRequest string` with tag `json:"pull_request,omitempty"`, beside `Profile`.
`normalizeTask` refuses a `PullRequest` other than empty, `on` and `off` with `entry %q has pull_request %q, and pull_request is on, off or empty: correct it, or set "" to clear it`; it runs before the project branch, so a task and a project take the field alike.
`UpsertPayloadKeys` adds `pull_request` to the optional keys, so every upsert path accepts it; `replayFields` gains the `pull_request` entry after `profile`, so a replay merges it.
`scopeChanged` compares `PullRequest`, so the run lock refuses a change to the field on a held entry, which fixes the field while a run holds the entry; its doc comment names the field among the compared ones.
`storeVerbs`' upsert `Use` line and field list gain `"pull_request" string — "on" or "off" overrides the receiving branch's beam.yaml pull_request mode; "" clears it`.
Tests: `TestNewTask` gains a subtest that `pull_request` `on` and `off` round-trip in the JSON, empty stores no key, and `maybe` is refused naming the accepted values;
`TestRunLock` gains a row in which a held entry's `pull_request` change is refused with the run-lock error while its status change still passes;
`TestHelpSchema_LeafCommands`' upsert case gains `pull_request` in `mustContain`.
The package doc gains a `# Pull request` paragraph: the field's values, that it is scope for the run lock, and that the landing reads it against the parent's beam.yaml mode.

**ImpactSummary:** One new optional board field threaded through validation, replay, the upsert allowlist and the run lock; no existing field changes and board.json written before it reads unchanged.

**Commit:** 3: board: pull_request field on tasks and projects
