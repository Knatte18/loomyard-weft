---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/slackengine/outbound.go:96-133
    origin: lens:code-correctness
    summary: the one line naming how many mail messages older than 24 hours were skipped is never posted when a later message of the same read fails, because the skip count is a local variable and the skipped ids are already in the outbox record on the redelivered read
    fix: In internal/slackengine/outbound.go keep the skipped count in outboxRecord (add a field such as `Stale int` with `json:"stale,omitempty"`), increment it and write it with the record in the same writeOutbox call that records the skipped id in Handled, and post the summary line from that persisted count when the read's handler finishes, including before the Slack-failure return at line 122 and the writeOutbox-error return at line 128. Clear the count with the rest of the record after a confirmed acknowledgement. Add a TestOutboundRead row where a stale message is followed by a fresh message whose post fails, then a second read succeeds, and assert the summary line is posted once with the right count.
  - id: F2
    severity: LOW
    class: design
    location: internal/creelcli/goclient.go (HubStrands)
    origin: lens:code-tests
    summary: HubStrands, which every pane refusal of lym slack relies on to read the hub's strands, has no test that would fail if it listed a different strand set than the creel daemon does
    fix: Add a row to internal/creelcli/cli_test.go that builds a hub with known strands and asserts HubStrands(hubPath) returns the same set the daemon's delivery lists.
  - id: F3
    severity: LOW
    class: design
    location: internal/slackcli/wiring.go (receiverPaused)
    origin: lens:code-tests
    summary: receiverPaused picks the prime's pause for the hub's own orch and the beam's own worktree for a beam orch, and no test fails if it picks the wrong worktree
    fix: Add one row per receiver kind (hub orch, beam orch) to internal/slackcli's tests over a hub fixture with a pause record on one worktree only, asserting receiverPaused reports covered for that worktree's orch and not for the other.
  - id: F4
    severity: LOW
    class: design
    location: internal/slackengine/outbound_test.go:119-139
    origin: lens:code-tests
    summary: D5 counts a message Slack refused 5 times as handled whether or not the replacement line posts, but the only row covers a line that posts
    fix: Add a row to the test of the 5-refusal replacement line where the post of the replacement line itself fails, and assert the read is still acknowledged, the message is not retried and the record holds no refusal count for it afterwards.
  - id: F5
    severity: LOW
    class: scope
    location: internal/configcli/configcli.go:467-470
    origin: lens:code-affected-set
    summary: the diff rewrites the editor-resolution paragraph of the lym config help text, which no decision or card of the Slack relay licenses
    fix: Revert the hunk in buildConfigLong in internal/configcli/configcli.go to its previous wording, unless a gate required the change, in which case record that reason in the commit message.
---

### [MEDIUM] Stale-mail summary line lost when a read fails after the skip

**Location:** internal/slackengine/outbound.go:96-133
**Issue:** `post` counts mail older than 24 hours in the local `stale` and posts the summary only from `reportStale`, which runs at the budget return and at the end of the loop.
The Slack-failure return and the `writeOutbox` error return skip it.
Stale mail has the lowest ids, so it is handled first and its ids are written to `record.Handled` before a later message can fail.
On the redelivered read those ids hit the `continue` and are never counted again, so D5's "one line names how many it skipped" and its acceptance criterion are not met on that path.
A failed post of the line is also only logged and the count dropped.

### [LOW] HubStrands has no test

**Location:** internal/creelcli/goclient.go (HubStrands)
**Issue:** The pane check of every `lym slack` verb reads the hub's strands through `HubStrands` (`internal/slackcli/wiring.go`), and D6 requires that set to be the one creel's sender resolution reads.
`TestSlackCLI` supplies strands through fakes, and no test names `HubStrands`.

### [LOW] receiverPaused worktree choice is untested

**Location:** internal/slackcli/wiring.go (receiverPaused)
**Issue:** The relay's "the hub is paused" answer depends on `receiverPaused` choosing the prime for the hub orch and the beam's worktree for a beam orch.
No row fails if the two were swapped.

### [LOW] Replacement-line failure path of the 5-refusal drop is untested

**Location:** internal/slackengine/outbound_test.go:119-139
**Issue:** `refused` returns dropped true whether or not the replacement line posts, as D5 says, but only the posting case has a row.

### [LOW] Unrelated help-text rewrite in configcli

**Location:** internal/configcli/configcli.go:467-470
**Issue:** The hunk changes the wording of the editor paragraph in `lym config`'s long help.
`internal/configcli` is outside the record's scope list and the Slack decisions say nothing about it.

## Rejected

- Lens code-affected-set F2 (login and setup appear in the orch's command index): `clihelp.Audiences` is a closed set of operator, role and internal, and the orch's index renders the operator audience, so D9's wording cannot be met by a different annotation; no defect in the diff.
- Lens code-tests F-T3 (StopRelay and RelayLive reach `runproc.EndRole` with no test): `StopRelay` is a one-line call into `runproc.EndRole`, whose sequence and sparing of the caller are covered in `runproc`; a test of the wrapper would be a per-helper test.
- Lens code-tests F-T5 (ensureSlackRelay gave-up logging): the fork did not read the assertions and wrote that the row probably covers it, so the finding rests on suspicion.
- Handler check of the PATTERN edits to `PATTERN-config-layers`, `PATTERN-cwd-resolution` and `PATTERN-daemon-wakeups`: each only adds `slackengine` or its relay loops to a roster that a pinned test in `cmd/lym` enumerates, and the decision record's Constraints license the daemon-wakeups list; no finding.
- Handler check of the large `orchengine` `watcher_test.go` and `watcher_compact_test.go` changes: the diff folds the presence-slack case into the existing tests as a `slack` row or a two-case loop, and the reindentation comes from wrapping the body in `t.Run`; not an unlicensed refactor.
- Handler check of the declared PATTERN changes: `PATTERN.md` carries both declared entries with the declared text, the two background files carry the double-fork exception and the Slack boundary, and no top-level `PATTERN-reap-children` or `PATTERN-slack-boundary` entry exists; all landed.
- Handler check of commit messages: every commit in the range is a bare `N: card-name` with no `Pattern-Departure:` trailer, so none needs judging.
