```yaml
lens: interactions
outcome: reported
```

Pairs checked:

- D3 `lym slack stop` × D3/D6 prime-dobby restart while presence `slack` — **broken** (F1).
- D3 "started detached … outside every pane tree" × D6 `away` waits for first read — **broken** (F2).
- D5 whole-read ack/release × D4 backoff reset on outbound post (and D5 rationale: mail queues while relay down) — **broken** (F3).
- D4 relay Slack answers × D5 rationale "every outbound path goes through creel's journal" — **broken** (F4).
- D6 verbs × D6 acceptance criterion "answer in Slack" — **broken** (F5).
- D3 first creel read × D6 `away` wait × D8 refused post — held (first read makes `SLACK` a receiver before `away` sets `slack`; a pre-read post is logged and retried).
- D3 single-instance lock × D3's four starters (bring-up, `away`, `start`, dobby) — held (live → no-op).
- D3 re-exec × D4 cursor / D5 reserved read — held (dropped connection releases reserved read unacked; post-then-cursor order bounded).
- D2 incomplete config × D3 starters and D6 `away` — held.
- D2 one-machine-per-channel bound × D3 dobby restart — held (only owner machine runs the prime dobby).
- D4 `@<slug>` route × D5 answer hint — held.
- D4 pause answer × D5/D7 — held.
- D4 Slack presence texts × D6 `away` relay wait — held (relay reading the text is live).
- D5 sender filter × D7 orch replies/mirroring — held.
- D5 sender filter × D8 dobby reminders — held (dobby is a program of the hub).
- D5 sender filter × D6 notices / `CREEL` sender notices — held (`SLACK` mails nothing expecting a reply).
- D6 presence × D7 orch rules and D6 resume pointer — held.
- D6 presence × D8 reminders gated on `slack` — held.
- D6 `back` × D3 relay lifetime — held (relay keeps running while `terminal`, which D7's reply-only rule needs).
- D4 cursor × AC "exactly once across a restart after the cursor write" — held.
- AC D5 second-message failure × D5 release — held for one transient failure; F3 is the sustained case.
- Constraints (4 KiB cap, `daemonwakeups_test.go` list, no new dependency) × D3/D4/D5 — held.

Findings:

**F1 — MEDIUM — design — D3 `lym slack stop` × D3 dobby restart (decision-record.md lines 52–53; AC line 158).**
`lym slack stop` ends the relay through `EndRole`, but while presence is `slack` the prime dobby's next pass starts it again, so `stop` is silently undone within one dobby tick. The record states neither outcome (stop refuses / stop also sets `terminal` / restart is intended), and the acceptance criteria test `stop` and the dobby restart separately, never together. Evidence: decision-record.md:52–53; dobby tick cadence `internal/dobbyengine/doc.go` (probe tick, timer to a minute).

**F2 — MEDIUM — design — D3 "outside every pane tree" × D6 `away` wait (decision-record.md lines 51, 92; AC line 158).**
`proc.Detach` only sets `Setsid` (`internal/proc/proc_linux.go` `Detach`); the relay's parent stays the starter while the starter lives, and creel's `paneOfPeer` walks parent pids, not sessions (`internal/creelengine/identity.go` `paneOfPeer`, lines ~117–150). D6's `away` keeps the starter alive until the relay's first read, so that read resolves inside the starter's pane tree: from a non-orch strand's pane `resolveSender` refuses the `SLACK` program claim ("only an orch strand's pane may claim a program name") and `away` times out; `lym slack start` from such a pane races the starter's exit. The D3 acceptance criterion "creel takes its `SLACK` claim when `lym slack start` runs from a strand's pane" rests on this false premise.

**F3 — MEDIUM — design — D5 ack/release × D4 backoff reset (decision-record.md lines 65, 83, 86).**
D5 releases a whole read when one post fails and reposts every message before the failure; D4 returns the poll wait to 5 s on any outbound post, so each repost of the earlier messages resets the backoff D5 relies on. With a backlog of N messages piled up while the relay was down (D5's own rationale), or a message Slack keeps refusing at position k, the relay reposts 1..k−1 every cycle, never acknowledges, fills the channel with duplicates, and the backlog never drains. Unverified (from memory): Slack also limits `chat.postMessage` to about one message per second per channel, so a fast drain of a backlog itself triggers a 429 midway through every read. The record states no posting rate and no per-message acknowledgement. Evidence: decision-record.md:65, :83, :86–87.

**F4 — LOW — consistency — D5 rationale × D4 direct answers (decision-record.md line 86 vs lines 70, 72, 73, 93).**
D5's rationale says "every outbound path goes through creel's journal", but D4/D6 have the relay post to Slack directly (unknown slug, over-long body, paused hub, presence answers); none of those passes through creel. The rationale overstates the property.

**F5 — LOW — consistency — D6 decision × D6 acceptance criterion (decision-record.md line 93 vs line 159).**
D6 says only the Slack texts `away`, `back`, `status` are answered in Slack. The criterion reads "`back`, `status` and the Slack texts `away`, `back` and `status` set or print the presence and answer in Slack", which also asks the CLI verbs `lym slack back`/`status` to post to Slack — a behavior D6 does not decide.

Tool calls: none — the inherited context held everything this lens needed.
