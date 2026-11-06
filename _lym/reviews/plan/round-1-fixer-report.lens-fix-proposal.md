```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (fix-proposal lens).

I built cases against F1 to F6. F1, F5 and F6 hold. For F1, a delivered `outbound` line starts no worker, and step 3's clash check still runs ahead of the guard. F4's skipped interval adds no new blocking, because `state.UpdateJSON` already takes the same write lock. Three LOW findings remain.

**1. LOW — F3, card 9 Intent: "The Replies section of `internal/reportengine/doc.go` and the `RoleReplies` field comments name the `inbound` and `outbound` counts."**
- Summary: the proposal's doc set misses `ComputeReplies`' own doc comment, which goes false under the same change.
- Evidence: `internal/reportengine/replies.go:27` says "A reply counts only when its ReplyTo names a message the entries show as marked".
- Card 9 counts an `outbound` entry by `Answers`, and that entry's `ReplyTo` is `original.SendingID`, an id on the other hub. So a counted reply's `ReplyTo` names no marked message here.
- `ComputeReplies` is on the Edit list, but the Intent does not ask for this comment to change.

**2. LOW — F2, card 8 row: "A `hubFake` answering `OK: false` with a set `Code` and an `Error` holding a quote and a line break is retried…"**
- Summary: the row as proposed can pass without reaching the coded branch it exists to cover.
- Evidence: `internal/creelengine/client.go:248-254` switches on only `CodeStrandClaimOutsidePane` and `CodeProgramClaimInStrandPane`. Any other `Code` falls through to the plain `errors.New(resp.Error)`.
- A row using an arbitrary code therefore exercises the plain refusal, which the row above it already covers. It leaves the `codedRefusal` wrap untested through `attempt`.
- The row should name one of the two constants (`internal/creelengine/protocol.go:39,42`).

**3. LOW — F4, card 1: "after which `lym creel inbox` in each hub records it again (card 4)", against card 4: "so card 1's corrupt-record way forward holds at once".**
- Summary: the proposal covers only a hub whose last record attempt failed, but a corrupt record can be seen with no failed attempt, so card 1's unqualified sentence is still false inside the minute.
- Case:
  1. At t=0 a verb records successfully.
  2. At t=10 the file goes corrupt.
  3. At t=30 a `post` is refused by `Lookup`, or `lym creel hubs` refuses through `Live`. Neither is a record attempt, and `recordHub` skips inside the interval, so the last record still reads as a success.
  4. At t=40 the operator removes the file.
  5. At t=45 `lym creel inbox` records nothing.
- This is the review's own scenario. Card 4's second clause, "past `hubRecordInterval`", admits it, but card 1's way forward text does not, and "holds at once" reads wider than that clause. The entry returns by itself past the minute.

Tool calls:
- `grep` of `client.go`, `internal/state` and `internal/lock`: my context held only `client.go` lines 200 onward, and neither the connect path nor the lock timeouts.
- `grep` of `internal/reportcli`, `replies.go`, `docs/overview.md` and the `Code` constants in `internal/creelengine`: my context lacked the code constants and any other reply-count text.
