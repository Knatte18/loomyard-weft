```yaml
lens: code-correctness
outcome: reported
```

Task: review the code-correctness lens (each changed function or branch, one case run, correct or not).

**Per-function list**

- `receiverLiveness`, live listed strand: returns `true` and an empty warning. Correct.
- `receiverLiveness`, dormant listed strand: returns `false` and `lym reed resume in <Worktree>`. Correct. An empty `Worktree` would give "resume in " with nothing after it; that is a fixture edge, not a real path.
- `receiverLiveness`, unlisted orch of this hub: returns `false` and `lym beam start <slug>`. The prime's orch has an empty slug, so `cmp.Or` gives `root`. Correct.
- `receiverLiveness`, unlisted name of another hub, or a program: returns nil. Correct.
- `notLiveWarning`: `agentname.Parse` is followed by `MatchesRole`, so a beam orch gets the beam command and any other strand gets `lym reed resume`. Correct.
- `storePost` failure path: returns `posted` and the error before computing liveness, so a failed store reports no liveness. Correct.
- `storePost` race: strands are read before `s.do`, so the liveness value can be stale by one listing. It is the same listing the receiver check uses, and the message still queues. Acceptable.
- `post`, interrupt then store failure: returns `PostResult{Interrupted: true}` with the error, which matches the old behaviour. Correct.
- `post`, other-hub path: returns `ID` and `Waiter` with nil `ReceiverLive`. Correct.
- `PostInterrupt`, `Post` and `PostWait` wrappers: they pass the same fields through as before. Correct.
- `dispatch` `VerbPost`, plain post: sets `ReceiverLive` and `Warning` on the single response. Correct.
- `dispatch` `VerbPost` with `Wait`: builds a `postStamp`, and the other `awaitOutcome` callers pass an empty `postStamp{}`. Correct.
- `awaitOutcome`, warning ordering: the warning is written as a progress response straight after the pending one and before `watchClose`. If that write fails it returns nil, as the pending-write failure does. Correct.
- `awaitOutcome`, normal final response: carries `ReceiverLive`. Correct.
- `awaitOutcome`, reading path (`server.go:461`): builds `Response{...AckRequired: true}` without `ReceiverLive` and without `Interrupted`. See F1.
- `Client.exchange` (`client.go:283`): any `Progress` response goes to the `progress` callback, so the new warning progress reaches the CLI. Correct.
- CLI `warn`: a plain post prints once from `warn(resp)`. A waiting post prints once from the progress callback, because its final response carries no `Warning`. Correct, with no double print.
- CLI envelope: `receiver_live` is added only when `ReceiverLive` is non-nil, so the existing JSON shape is unchanged for programs and other hubs. Correct.
- Client reconnect (`client.go:213`): a dropped waiting post resumes as `VerbWaitReply`. The resumed pending and final responses carry an empty stamp. See F2.

**Findings**

- **F1, LOW, design**, `internal/creelengine/server.go:461`.
  - A `post --wait-reply` whose outcome is delivered through the `Reading` path (`AckRequired`) answers without `receiver_live`.
  - The CLI envelope then omits the field on that outcome, while the pending response and the other outcomes carry it.
  - The fix is to set `ReceiverLive` (and `Interrupted`) on that response from `stamp`, or to document that only the pending and plain outcomes carry it.
  - I did not check whether this path is reachable for a post's own reply outcome, so the severity is hedged.
- **F2, LOW, design**, `internal/creelengine/client.go:213` with `server.go:186`.
  - After a reconnect, a waiting post is resumed as `VerbWaitReply` with `postStamp{}`, so the final response has no `receiver_live`.
  - The CLI envelope drops the field after a reconnect.
  - The warning is not replayed, which is right because it was already printed.
  - The fix is to have the CLI keep `ReceiverLive` from the first pending response, or to accept and document the omission.

Tool calls (3):
1. Read `internal/creelengine/pool.go` 205-285, for `findStrand`, `isHubOrchName` and `checkReceiverLocked` (not in the manifest).
2. Read `internal/creelengine/client.go` 130-240, for `CallWatching`, `resume` and `exchange` (not in the manifest).
3. Read `internal/creelengine/client.go` 239-328 and `server.go` 440-480, for the rest of `exchange`'s progress handling and the final-response and ack paths (not in the manifest).
