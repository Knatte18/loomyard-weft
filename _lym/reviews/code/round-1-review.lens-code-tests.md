```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, the test that pins it.

**Per behaviour: pinning test**

1. A listed live strand answers `ReceiverLive` true with no warning.
   - Pinned by `TestStore_Mail` row `PostWith_AnswersWhetherTheReceiverIsLiveAndHowToStartOneThatIsNot` (`verbs_test.go`, receiver `alpha`).
   - Also pinned at the protocol level (`server_test.go`, live post) and the CLI level (`cli_test.go`, `liveStderr == ""`).
2. A listed dormant strand answers false and a `lym reed resume in <worktree>` warning.
   - Pinned by the same `verbs_test.go` row (`beta`), the server row (warning prefix) and the CLI test (exact stderr line).
3. A dormant prime orch warns `lym beam start root`.
   - Pinned by the `verbs_test.go` row (`orch`).
4. A dormant beam orch warns `lym beam start payments`.
   - Pinned by the `verbs_test.go` row (`dormantBeamOrch`).
5. An unlisted orch of this hub answers false with `lym beam start gone`.
   - Pinned by the `verbs_test.go` row (`absentBeamOrch`).
6. A program receiver answers nil and no warning.
   - Pinned by the `verbs_test.go` row (`batten`).
7. A receiver on another hub answers nil via `postOtherHub`.
   - Pinned by the `verbs_test.go` row (`zz:orch`), assuming `withHubs` registers `zz`. Existing tests use the same pair.
8. A not-live receiver still queues the message.
   - Pinned for `beta` only, by `wantBodies(... beta ... "hi")` in the `verbs_test.go` row.
   - The orch cases are not checked for queueing. Low risk, since the queueing path is unchanged.
9. A non-waiting post puts `ReceiverLive` and `Warning` into the daemon response.
   - Pinned by the `server_test.go` row `PostAnswersWhetherItsReceiverIsLive...` (live and dormant `mustCall`).
10. A waiting post sends the warning as a progress response right after the pending one.
    - Pinned by the same server row (`nextProgress`, `note.Warning`).
11. A waiting post to a live receiver sends no progress warning (`stamp.warning != ""` guard).
    - **None.** The waiting call in the server row targets the dormant `beta` only.
12. The pending response of a waiting post carries `ReceiverLive`.
    - **None.** The server row never asserts it.
13. The final outcome response of a waiting post carries `ReceiverLive`.
    - **None.** The waiting call in the server row ends in a client timeout, so the outcome branch at `server.go` (`resp := Response{... ReceiverLive: stamp.receiverLive ...}`) is never reached.
14. `--wait-reply` in the CLI prints the warning once and puts `receiver_live` in the envelope.
    - **None.** The CLI test uses plain posts only.
    - Nothing pins the `CallWatching` progress callback or the `warn(resp)` call after it. Nothing checks that a waiting post is not warned twice.
15. The CLI omits `receiver_live` when the daemon answers nil (program or other hub).
    - **None** at the CLI level. Only the engine pins nil.
16. `--parent` and `--interrupt` posts carry `receiver_live` in the envelope.
    - **None** at the CLI level. They share the plain-post code path, so this is low value.
17. The `PostWith` signature change to `PostResult`, including the `Interrupted` stamp on store failure after an interrupt.
    - Pinned by the existing interrupt rows in `TestStore_Mail`, which the diff only adapts (`deliver_test.go`, `outbound_test.go`, `verbs_test.go`).
18. The `PostInterrupt`, `Post` and `PostWait` wrappers.
    - Pinned by existing tests.

**Findings**

- **LOW, scope.**
  - Location: `internal/creelengine/server.go` (`awaitOutcome`, the pending and final `Response` literals).
  - Summary: the waiting-post behaviours with no row are: `ReceiverLive` on the pending response, `ReceiverLive` on the final outcome, and no warning progress for a live receiver.
  - Evidence: the waiting call in `server_test.go` (row `PostAnswersWhetherItsReceiverIsLive...`) only targets the dormant `beta` and ends by timeout.
  - Fix: extend that row, or add assertions to it, with a waiting post to a live receiver that gets a reply. Assert no progress note arrives and that the final response has `ReceiverLive` true.
- **LOW, scope.**
  - Location: `internal/creelcli/cli.go`, the `post` command's `warn` and `CallWatching` wiring.
  - Summary: no CLI test covers `--wait-reply` with a not-live receiver. The warning could be dropped or printed twice, and `receiver_live` could be lost from the envelope, without any test failing.
  - Evidence: the only new CLI test in `cli_test.go` runs plain posts.
  - Fix: add a `--wait-reply` case to the new `TestCreelCLI` subtest, or a second subtest, against the dormant strand with a short timeout. Assert stderr holds the warning exactly once.
- **NIT, scope.**
  - Location: `internal/creelcli/cli.go`, `if resp.ReceiverLive != nil`.
  - Summary: the omit-when-unknown branch is not covered at the CLI level.
  - Evidence: the new CLI test only posts to listed strands.
  - Fix: assert in the new CLI subtest that a post to a program receiver has no `receiver_live` key, if a program receiver is cheap to set up in that rig.

No test-economy duplication found. The three layers (store, protocol, CLI) each assert different things, and the warning wording is asserted exactly at two layers, which is acceptable.

**Tool calls:** none. The inherited context already held the diff and the test bodies.
