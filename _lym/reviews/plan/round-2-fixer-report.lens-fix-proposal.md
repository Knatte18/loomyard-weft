```yaml
lens: fix-proposal
outcome: reported
```

Task: check the reviewer's proposals the fixer carried out (F1's reset and write order, F2, F3, F4), using the fix-proposal lens.

**MEDIUM: under F1, a re-exec erases the evidence the starter's reset relies on, so failures that are not in a row can still end in `gave-up`.**
- Changed text: card 09, the bullet "The starter owns the reset: a due `whileAway` start that finds a `FirstReadAt` later than `LastSpawnAt` sets `StartFailures` to 0 … so only failures in a row end in `gave-up`."
- The case:
  1. Four failures are counted.
  2. The fifth spawn reaches an accepted first read and runs for days. No starter runs in that time, so `StartFailures` stays at 4.
  3. The binary is replaced. `CheckBinary` re-execs the relay, which runs from step 2 again, and step 3 writes `FirstReadAt` zero ("its pid and start time, `FirstReadAt` zero and no refusal").
  4. The new image dies before its first read.
  5. The next due start finds `FirstReadAt` zero and counts the fifth failure, so it answers `gave-up`.
- Two failures days apart, separated by a healthy relay, end the restarts. That contradicts the sentence's "only failures in a row" claim.
- The review's text "Any relay's accepted first read resets `StartFailures`" did not have this gap. A reset that only the starter makes, and only lazily, does.
- One remedy: the relay's step 4 write of an accepted `FirstReadAt` also sets `StartFailures` to 0 in the same read-modify-write. Another: step 3 keeps `FirstReadAt` on a re-exec.

**MEDIUM: under F3, a lasting `Hub.Beams` error blocks the channel with no bound and no word to the operator.**
- Changed text: card 07, step 3: "A `Hub.Beams` error … the poll returns it with the cursor left before the message, and the relay retries with its backoff."
- `Hub.Beams` comes from `hubgeom.DobbyStart(prime).Beams` (`internal/hubgeom/dobbygeom.go:105`, `localBeams`). That listing can fail on every read, for example on an unreadable origin record.
- With the cursor held, every later message waits behind the `@slug` message. That includes `away`, `back` and `status`. The operator gets no answer, and the new text names no Warn log.
- A failed post is bounded: after 5 minutes it is answered as not delivered and passed. This branch has no such bound, so one `@slug` message can stop the relay forwarding anything, silently.
- Remedy: give the `Hub.Beams` error the refused post's path: Warn once per episode, then after 5 minutes a Slack answer and the cursor moved past the message.

**LOW: under F3, "read only for a text opening with an at-sign slug" depends on an order the card does not state.**
- Same changed text as above.
- `route(shortname, text string, beams []string)` takes the slugs already read, and nothing else in card 07 says the poll reads `Hub.Beams` only when the text opens with `@`.
- An implementation that reads the beams before every `route` call fails on every message whenever `Hub.Beams` errors.
- Remedy: state in the route bullet that the poll calls `Hub.Beams` only for a text opening `@&lt;slug&gt; `.

**F2 and F4: no case found.**
- F2: `goCallTimeout` bounds every other `GoClient` call at `goclient.go:88–136`, so exempting `ReadWait` alone is right.
- F4: the root `Long` is about 944 bytes, far under `maxLongBytes` = 2000 (`clitree_test.go:167`), so adding `slack` breaks no cap.

Tool calls:
1. grep of `cmd/lym` and `internal/hubgeom`: the `Long` byte cap and where `Beams` comes from. My context showed neither.
2. Byte count of the root `Long` in `cmd/lym/main.go`: needed to check F4 against the cap. My context held only the modules line.
