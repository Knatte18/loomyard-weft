```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I named the test row that fails if it breaks, or none.

**Behaviour → pinning test**

1. **`Store.AwaitsReply` counts an open question a strand holds.** Pinned by `TestStore_AwaitsReply/AQuestionAStrandHoldsUnansweredIsAwaited`.
2. **`Store.AwaitsReply` counts an unread reply.** Pinned by `TestStore_AwaitsReply/AnAnswerTheAskerHasNotReadIsStillAwaited`.
3. **A reply the asker has read ends the wait.** Pinned by `TestStore_AwaitsReply/AnAnswerTheAskerReadEndsTheWait`.
4. **A plain message is not a question.** Pinned by `TestStore_AwaitsReply/APlainMessageIsNoQuestion`.
5. **A question a program holds is not counted** (the `isStrandName(msg.Holder)` clause). Pinned by `TestStore_AwaitsReply/AQuestionAProgramHoldsIsNeverChasedSoItIsNotCounted`.
6. **Another strand's question is not counted** (the `msg.From == strand` clause on `asked`). Pinned by `TestStore_AwaitsReply/AnotherStrandsQuestionIsNotTheAskers`.
7. **A question settled unanswered ends the wait.** Pinned by `TestStore_AwaitsReply/AQuestionSettledUnansweredEndsTheWait`.
8. **An empty strand is refused, naming the way forward.** Pinned by `TestStore_AwaitsReply/AnEmptyStrandIsRefused`.
9. **A question returned unread is not counted** (the `msg.EndedUnreadBy == ""` clause). **None.**
10. **The reply side checks `original != nil` and `original.From == strand`.** **None:** no row has the strand holding unread mail with `ReplyTo` 0, or holding a reply to someone else's message.
11. **The daemon dispatches `awaits-reply` and reads the strand from `From`.** Pinned by `TestDaemon_AwaitsReplyAnswersThroughTheProtocol`, true and false.
12. **`GoClient.AwaitsReply` and `AwaitsReplyProbe` work, signed as a program.** Pinned by the `TestCreelCLI` subtest "the reply probe reports a strand's unanswered question…". It calls the probe, which calls `GoClient.AwaitsReply`. The reach report misses this only because it could not classify `TestCreelCLI`.
13. **An ungated Done is held while the probe answers true, and the run ends at the turn end after the reply.** Pinned by `TestRun_Wait_ReplyHold` row 1 (`wantSteps` 4).
14. **A gated Done is held, and no gate runs during the hold.** Pinned by row 2 (`gateCallsAtReply == 0`, `wantSteps` 4).
15. **Two quiet probes release the hold**, ungated finalizing done and gated evaluating its gate. Pinned by rows 3 and 4 (`wantSteps` 10).
16. **A probe error reads as "awaits nothing".** Pinned by row 5 (`wantSteps` 0).
17. **A run with no strand is never probed.** Pinned by row 6.
18. **The probe is asked only about the run's own strand.** Pinned by the `asked` loop in every row.
19. **Any turn start or event read during a hold ends it** (`wait.go:332-334`, `turnStartSeen` in `pollEventsTick`). **None.** These are mutation survivors `wait.go:333`, `:969` and `:986`.
20. **After a release, a gated evaluation that does not finish the run leaves no hold behind** (`replyhold.go:75`). **None.** This is survivor `replyhold.go:75`.
21. **The hold under a pause, and the hold when a pause lifts** (`releaseReplyHold`'s `run.covered` branch, and `holdForReply` in the `boundaryHeld` branch). **None.**
22. **The hold sends no notice.** **None.**
23. **The four CLIs (loom, burler, webster, batten) wire the probe into their runners.** **None.** `TestWire_EverySeamFilled` checks `c.env` seams, not the runner's unexported `replyProbe`.
24. **Probe-failure logging** (`failures.Fail` and `failures.Recover`, survivors `replyhold.go:36` and `:39`). Log-only, so no finding.
25. **The `parent-directive-*` stencils and `driftengine` mode text.** Static prose, exempt.

The test values are independent literals: step counts, booleans, `OutcomeDone`. I found no tautological test and no redundant test. `TestStore_AwaitsReply`, `TestDaemon_AwaitsReplyAnswersThroughTheProtocol` and the creelcli subtest each reach different code.

**Findings**

- **MEDIUM, `design`, `internal/shuttleengine/replyhold_test.go:562`.** No test checks that a turn start read during a hold ends it.
  - The contract: `wait.go:45-48`'s doc and the summary say "Any event or turn start read after that ends the hold", so a release cannot finish the run in the middle of the reply's turn.
  - Why the test misses it: `TestRun_Wait_ReplyHold` drives `fakeEngine`, which has no `ParseSessionSignals` (`fakes_test.go:402`), so `turnStartSeen` is never true.
  - The reply row adds the turn end at the same step the probe goes quiet, so Done re-probes and finishes either way. Survivors `wait.go:333` (the dropped `run.replyHold = nil`), `:969` and `:986` confirm this.
  - Missing row: an engine that parses session signals; the reply writes a turn start and then no turn end for more than `replyQuietProbes × gateRecheckEvery`. Assert that Wait has not returned until that turn's end.
- **MEDIUM, `design`, `internal/shuttleengine/replyhold.go:75`.** No row has a gated release whose evaluation leaves the run going: a gate that fails with budget left and re-prompts, or that returns pending.
  - Dropping `run.replyHold = nil` survives (survivor line 75), because the gated rows' gate passes at once.
  - If the hold stayed set, the `run.replyHold != nil` branch would win over the pending and unsent-reprompt branch. It would call `handleGatedBoundary` again every `gateRecheckEvery`.
  - Missing row: a failing gate with `Attempts` above 1. Assert one re-prompt per turn end, not one per 5 s.
- **LOW, `design`, `internal/creelengine/awaitsreply_test.go:180`.** `TestStore_AwaitsReply` has no row for a question returned unread.
  - The `msg.EndedUnreadBy == ""` clause in `verbs.go:1095` is unpinned.
  - Without that clause, a returned question (now held by the asker, a strand, `ExpectReply` set and never `Replied`) counts forever. It would hold the asker's run until the deadline.
  - Missing row: `releaseGoneHolder` or a journal of `posted` then `returned`, with want `false`.
- **LOW, `design`, `internal/creelengine/verbs.go:1096-1097`.** No row has the strand holding unread mail that is not a reply to its own message.
  - Neither the `original != nil` guard nor the `original.From == strand` check on the reply side is pinned.
  - Missing row: alpha holds unread plain mail, want `false`, which exercises `ReplyTo` 0.
- **LOW, `design`, `internal/shuttleengine/replyhold.go:502-505` and `wait.go:777`.** The hold's pause paths are untested:
  - a release while covered, which sets `boundaryHeld`;
  - `holdForReply` when a pause lifts.

  Missing row: a gated run with a pause-record fixture covering the release tick.
- **LOW, `design`, at the `runner.SetReplyProbe` lines:** `internal/loomcli/wiring.go:714`, `internal/burlercli/wiring.go:152`, `internal/webstercli/wiring.go:204`, `internal/battencli/drive.go:518`.
  - Nothing fails if any of the four wiring lines is dropped. The probe is what makes the board entry's fix reach real runs.
  - `TestWire_EverySeamFilled` (`internal/loomcli/wiring_seams_test.go:182-209`) checks only `c.env` seams.
- **NIT, `design`, `internal/shuttleengine/replyhold_test.go:624`.** The doc says the hold sends no notice, but no row sets a notifier and asserts it stays uncalled during a hold.

Tool calls:
1. Read `TestWire_EverySeamFilled` in `internal/loomcli/wiring_seams_test.go`, to see whether it pins the runner's reply probe; the manifest named it only from the reach report.
2. Read `fakeEngine.ParseEvents` in `internal/shuttleengine/fakes_test.go`, to see whether the engine `TestRun_Wait_ReplyHold` uses can produce turn starts; the manifest did not cover the fake.
