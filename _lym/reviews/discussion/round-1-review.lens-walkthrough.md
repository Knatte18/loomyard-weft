```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's purpose.

## Decisions walked

- **D1**: purpose met. Case: an agent in pair P runs `post --parent`, P's parent is beam B, and B's orch is dormant. `NotifyParent`'s walk calls `ResolveParent(P)` to get B, `NearestLiveOrch` skips B and lands on the prime, and `OrchStrandOf(prime)` gives the prime orch, which is posted to. A prime with no orch strand gives `recorded=false`, which leads to the refusal.
- **D2**: purpose met. Case: a Burler in a pair asks through `--parent` and the nearest live orch answers. The sender stamp is an orch strand, so the widened trust covers the answer.
- **D3**: purpose met. Case: a loom-spawned agent in pair P is held. `loomcli/wiring.go:464` calls `NotifyParent(location)` for the same pair cwd that `--parent` starts from, so both reach the same strand.
- **D4**: purpose met. Case: the LLM driver started by `lym loom start` in the pair runs `--parent`. That walks from the pair, as `notifyRunParent` does when the pair is present, so batten's announcement and the driver's message go to the same orch.
- **D5**: purpose met. Case: the chair runs `post --expect-reply &lt;advisor&gt;`, the advisor runs `post --reply &lt;id&gt;`, and `Reply` addresses the answer to `original.From`, the chair.
- **D6**: purpose met. Case: an advisor's answer reaches the chair as `[creel]` mail from an agent strand. The new sentence leaves it to the seat directive, which treats it as advice.
- **D7**: purpose met. Case: a seat table with more than one seat. With the append at `seatengine/prompt.go:166` removed, the spec requires nothing peer-related.
- **D8**: purpose met. Case: the operator answers a held pane-mail strand with `lym creel post &lt;strand&gt;`. An EXTERNAL sender's post is not a notice (`isNotice` requires a program sender), so it is reminded and the agent reads it. The answer arrives by another route, but it arrives.
- **D9**: purpose met. Case: a `--wait-reply` post is marked, read and answered by the holder through `Reply`, which checks `original.Holder`.
- **D10**: purpose broken in three cases; see findings 1–3.
- **D11**: purpose met. Case: `log --id` of a message that was chased twice.
- **D12**: purpose partly met. Case: counting replies per receiver role from `Log` output. A log entry has no link from a reply to its message; see finding 4.
- **D13**: purpose met. Case: the scan over the tree after the task. The only non-test Go literal naming `SendMessage` is `hold.go:57`, which D3 changes, and the webster line at 171 is allowlisted.

## Findings

1. **BLOCKING · design · D10 (holder-gone).** If the holder disappears after reading a marked message, the message is never settled.
   Case: holder H reads marked #N, then H's strand is removed, and H holds no other unread mail or queued inject.
   The removed-holder sweep only covers strands that hold unread mail or a queued inject (`internal/creelengine/sweep.go:179-191`), so it never looks at H.
   D10's clock "counts only while the holder is live", so it stops and no chase is posted that would give H unread mail.
   The "`unanswered` at once, `holder-gone`" step therefore never runs, and a `--wait-reply` sender waits with no end.
   Evidence: `sweep.go:184` checks `msg.Status == StatusUnread`; creel `doc.go`, "Removed holders": "A sweep runs while a strand holds unread mail or has an inject queued".

2. **MEDIUM · design · D10 (restart derivation).** A restart cannot rebuild a clock that pauses with the journal records D10 names.
   Case: H reads #N at T0, a pause covers H from T0+5m to T0+25m, and the daemon restarts at T0+30m.
   D10 rebuilds chase state from the read time and the `chased` records, which gives 30 minutes elapsed, while the frozen clock has counted 10. The first chase therefore goes 5 minutes early, against the 15-minute interval.
   Inject TTLs survive restarts only because `ageInjects` and `persistAged` journal the time used so far as `aged` records every `agedPersistInterval` (one minute). D10 names no such record.
   Its claim that inject TTL already stops under another sender's hold is also wrong: `ageInjects` only stops when the receiver is not live or a pause covers it.
   Evidence: `internal/creelengine/deliver.go:22`, `:555-575`, `:603-620`; `journal.go:90-92`.

3. **LOW · design · D10 (reply to a chase by a non-holder).** A forwarded chase's reply is lost.
   Case: H forwards an unread chase to strand X, and X runs `post --reply &lt;chase id&gt;`.
   `Reply` sends it to `original.From`, which is `&lt;shortname&gt;:CREEL`, with no receiver check. The reply sits as unread mail in a program mailbox that nothing reads and creel never reminds, so neither the original sender nor anyone else sees it.
   D10 calls this "an ordinary reply to the chase" without saying where it lands.
   Evidence: `internal/creelengine/verbs.go:180-200` (the `Reply` holder check and `appendPostedLocked(sender, original.From, kindReply…)`); creel `doc.go`: "A program's mailbox is never reminded".

4. **LOW · design · D12.** The report cannot tell replies to marked messages from other replies through the log call D12 names.
   `LogEntry` has no reply-to link back to the original message, so "the replies" can only be every reply a role posted.
   D9 adds the mark to the log entry, but the record does not say the reply link is exposed as well.
   Evidence: `internal/creelengine/fold.go:104-127` (no `ReplyTo`), against `journal.go:85` `ReplyTo` in `record`.

## Tool calls

1. `LogEntry` struct and sweep gating (`fold.go`, `sweep.go`): context lacked the log entry fields and the sweep's subject set.
2. `grep` in `verbs.go` for `Post`/`ReplyTo`: needed to find the reply routing.
3. Receiver-check grep in `verbs.go` (no output): looked for the receiver rule that applies to replies.
4. `verbs.go` `post`/`storePost`/`Reply` bodies: needed to see whether a reply to a program sender is checked as a receiver.
