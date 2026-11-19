```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I traced what reads, calls or describes it outside the changed lines, and checked whether each of those is still right.

**Per change: the set found, covered or not**

1. **`MachineBridge` gains `RelayPostReady`** (`internal/creelengine/crossmachine.go`).
   - It has two implementers, `slackBridge` (`internal/creelcli/slackbridge.go`) and `creelfake.Bridge`; both are updated, so this is covered.
   - It is wired at `server.go:50` and `server.go:97` (`Options.Bridge` → `SetMachineBridge`), which need no change: covered.
   - `creelengine/doc.go:804-806` describes the seam for machine mail only. The diff's new paragraph at `doc.go:105-110` names the relay use, so this is covered.
2. **A post to `&lt;shortname&gt;:SLACK` from a strand or EXTERNAL is refused, or answers `receiver_live` plus a warning** (`verbs.go` `post` and `slackRelayCheck`).
   - The Go callers that post to the relay are `slackcli/hook.go:188-192` (the slack turn hook) and `slackengine` (`bridgein.go`, `outbound.go`, `keeper.go`, `holder.go`, `relay.go`, `bridgechannel.go`).
   - The `slackengine` callers post as the program `&lt;shortname&gt;:SLACK` to orchs, not to the relay, so they are unaffected.
   - The turn hook is affected: it posts through `newStrandClient(strand)` as the orch strand (`hook.go:208-217`), not as a program. Its post now passes through `RelayPostReady`. See findings 2 and 3.
   - The CLI print path (`cli.go:452-473`) and the daemon paths (`server.go:316-320` and `awaitOutcome` at `server.go:471-476`) already carry `ReceiverLive` and `Warning`: covered.
   - The orch role file `contracts/orchrole/role-slack.md:7` describes this behaviour and is left wrong: finding 1.
   - The relay's own doc, `slackengine/doc.go:82` ("drops the mail of an orch whose button is off"), stays true for program mail and for a button switched off after the post was stored: covered.
   - The `PATTERN-agent-addressing` "Slack boundary" entry is unchanged in meaning: covered.
3. **`Response.ReceiverLive` and `Warning` doc comments** (`protocol.go`) and **`PostResult.ReceiverLive`** (`verbs.go`).
   - Readers are `server.go` and `cli.go:471`, plus the `creelcli/doc.go` envelope line, which the diff extends: covered.
4. **`creelfake.Bridge.Refuse` now keys both `SendReady` and `RelayPostReady`, with its parameter renamed.**
   - Its callers are `machinemail_test.go:309`, `machinemail_test.go:417` and `verbs_test.go:448`.
   - None of the pre-existing callers posts to `ly:SLACK`, so the shared map has no effect on them: covered.
5. **The `postCmd` Long text drops three facts** (idle wake, the mark shown beside the reply command, and the resolved `receiver` under `--parent`).
   - Each fact is still documented: the idle wake at `creelcli/cli.go:82` (the creel root Long), the mark at `creelengine/doc.go:202`, and the `receiver` key at `creelcli/doc.go:72`. Covered.
   - The cut was forced by the `maxLongBytes` cap in `cmd/lym/clitree_test.go:280`, whose message says to move long pages into the package doc.
6. **Commit messages, departures and PATTERN.**
   - No commit in the range carries a `Pattern-Departure:` or `Loosened-Check:` trailer, and none is needed: no test assertion at the review base was widened.
   - The directive declares no PATTERN changes.
   - `PATTERN-refusal-way-forward` binds only the webster, shed and loom packages, and the new creel refusals carry a `way forward:` clause anyway.
   - Covered.

**Findings**

1. **MEDIUM**, consistency.
   - Location: `contracts/orchrole/role-slack.md:7`.
   - Summary: the orch's Slack role file still says that mail posted to `&lt;shortname&gt;:SLACK` while the button is off "is dropped". After the diff, such a post is refused with a way forward. The file also never says that a post while the relay is down returns `receiver_live: false` with a warning. An orch reading it is told the old, silent behaviour the board entry set out to end.
   - Evidence: `role-slack.md:7`; `slackbridge.go` `RelayPostReady` (the button-off refusal); `verbs.go` `post` and `relayDownWarning`.
2. **MEDIUM**, consistency.
   - Location: `_lym/landing/summary.md`, the "What changes" section.
   - Summary: the description says lym's own program senders, "such as … the slack turn hook, keep their current behaviour". The turn hook is no program sender: it posts as the orch strand through `newStrandClient`, so its post now goes through `RelayPostReady`. The claim is not borne out by the diff.
   - Evidence: `internal/slackcli/hook.go:173`, `hook.go:188-192` and `hook.go:208-217`; `verbs.go` `slackRelayCheck` (skips only `KindProgram`).
3. **LOW**, design.
   - Location: `internal/slackcli/hook.go:181-195`, reached through `verbs.go` `slackRelayCheck`.
   - Summary: the turn hook reads the button itself, then posts. When the button turns off between that read and the post, the post is now refused rather than stored and dropped. The hook then returns an error and logs "the reply is lost to Slack", where before it took the silent drop.
   - The outcome is the same (nothing reaches Slack), but the hook now reports it as a failure. Neither the hook nor the diff handles that path, and no test covers a hook post through the new check.
   - Evidence: `hook.go:181-195`; `slackbridge.go` `RelayPostReady`.

Tool calls:
1. Searched for `MachineBridge` implementers, `SLACK` docs in contracts and plugins, and the removed help facts elsewhere. The manifest lacked the docs and stencils outside the diff.
2. Read `contracts/orchrole/role-slack.md`, the `SLACK` lines of `orch-role.md`, the relay paragraph of `slackengine/doc.go` and `creelengine/doc.go:795-812`, to check whether each doc still holds.
3. Searched for Go callers that post to the relay, and for `SLACK` in contracts and plugins beyond the orch role, to find the senders the new check reaches.
4. Read `slackcli/hook.go` `replyHook`, to learn the turn hook's post path.
5. Read `slackcli/hook.go` `orchClient`, to learn the turn hook's sender kind.
