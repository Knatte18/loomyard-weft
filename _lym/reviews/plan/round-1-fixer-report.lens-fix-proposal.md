```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out as proposed (F2, F4, F5, F6, F7, F8, F9, and F1 apart from the reassigned entries).

**LOW**: the F3 give-up text in card 11's `handle` step 2 doesn't cover an operator command written as a thread reply.
- **Changed text:** "When `runPeerCommand` fails, `operator` returns its error and neither cursor moves … Once the first failure is `retryWindow` old, it answers in the channel naming the failure and moves the operator cursor past the message."
- **Problem:**
  - Card 11 handles an operator reply in a thread "as above and answered with `PostThreadReply` in its thread", and reads threads "after each thread's cursor and the thread floor".
  - For a thread-reply `approve` whose `users.list` call keeps failing, the new text says the give-up goes "in the channel" rather than in the thread.
  - It also says the channel's operator cursor moves, when the cursor that holds a thread reply is that thread's cursor. So the thread case either answers in the wrong place or never moves past the message.
- **Evidence:** card 11 Intent, the line "An operator reply in a thread is handled as above and answered with `PostThreadReply` in its thread", against the new step 2 sentences. The new `TestBridgeReceive` row covers only a channel-level `approve`.

**Proposals that held up:**
- **F2:** `errors.Is` walks `daemonRefusal`'s chain, so `daemonRefusal.Is` answering true for `ErrDaemonRefusal` still lets coded refusals match their own sentinels through `Unwrap`.
- **F8:** the premise holds. `agentname.Parse("osl::zz:orch")` splits into 4 parts and fails (`internal/agentname/agentname.go:138-146`). `waitsForHubOrch` needs `IsProgram(msg.From)`, which an orch sender address doesn't satisfy (`internal/creelengine/sweep.go:188`). So the undelivered branch at `sweep.go:239` is taken without changing `releaseGoneHolder`.
- **F1, F4, F5, F6, F7, F9:** I could not build a case where any of them breaks a sibling card or row.

Tool calls:
- Read `isStrandName` and `waitsForHubOrch` in `internal/creelengine/sweep.go`, to test F8's premise against the gate before `releaseGoneHolder`; my context held only lines 220-249.
- Read `agentname.Parse`, to confirm that a `<machine>::<name>` sender fails `isStrandName`; my context didn't have the parser's body.
