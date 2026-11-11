```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision on one concrete case, end to end, and checked the end state against the decision's purpose.

Decision, case and result:
- D1, a task's `run_settings` sets `discussion_assist_reviewer_model: fable[high]` over a hub that sets none: the key is category none (`internal/loomengine/config.go:657`), so the task layer reaches the assistant seat. Purpose met.
- D2 case A, generation 1: round 1 asks, the parent answers with findings, Burler-Fix fixes them, the Bouncer continues, round 2 asks and the parent answers satisfied, so round 3 would not ask. Purpose met.
- D2 case B, a mid-perch goto back to Discussion-Write after a round that escalated and was never settled: no new generation starts, and the rewritten record reaches the next round of the same generation. That round asks only if the earlier parent answer held findings. Purpose "every rewritten record goes to the parent" is not met.
- D2 case C, a settled perch whose record changes, which starts generation 2: `startNextGeneration` archives the run directory and seeds round 1 again, so the round number restarts at 1. Purpose met if the generation number is read from the generation record, but the record's claim that round numbers carry on is false.
- D3 case A, a review row restarted in the same round with a new token: the ask record is found by generation and round, so no second ask is sent. Purpose met.
- D3 case B, the run's parent orch is not live: `NotifyParent` falls back to the prime's orch and posts, so the state is not `no-parent`. Purpose partly met: the round still goes on, through `timeout` at the deadline.
- D4, `--findings-file` from the orch fork while the ask is open: the guard passes, the report is written, the ask is set `done` and the copy is made once the table ends. Purpose met.
- D5, `review_timeout_min` is shorter than the room the cap needs: the record gives no cap formula and no answer for a cap with no room. Purpose ("the chair always outlives the wait") is not shown.
- D6, the run's driver is an LLM strand that runs `lym shed step` in its pane, so the review row posts the ask from that process: `NotifyParent` signs the mail as the driver strand, not `<shortname>:LOOM`. The orch-role section keyed on the LOOM sender does not match. Purpose at risk.
- D7, a parent report that holds a `decision`-class finding: the chair keeps it as class `design` with `origin: parent`, Burler-Fix fixes it, and it is not routed back to the parent. Purpose met.

Findings:

1. **MEDIUM, design, D2 (generations; "a goto back to Discussion-Write … starts a new generation").**
   A goto back to Discussion-Write on a perch that is not settled starts no new generation, so the rewritten record may never go to the parent.
   - Only a settled verdict whose subject changed starts a generation: `internal/shedadapters/bouncer.go:446-453` and `:785-826`.
   - The comment at `internal/shedadapters/bouncer.go:367-369` says a goto onto an escalated round that is still undecided "halts Awaiting again on that round, and a `continue` then sends the rewritten artifact to a fresh review round". That round is in the same generation.
   - `internal/shedadapters/bouncer.go:238` says "a goto or a resume restarts nothing".
   - D2's ask rule then skips the parent unless its last answer held findings, which misses D2's own purpose "every rewritten or amended record goes to the parent". The bracketed claim that a fresh `lym loom start` starts a generation is false for the same reason.

2. **MEDIUM, design, D2 and D3 ("while round numbers carry on"; where the generation is read from).**
   A new generation restarts round numbers at 1; they do not carry on.
   - `startNextGeneration` archives the run directory, recreates it empty and calls `seedCall`, which seeds round 1: `internal/shedadapters/amendment.go:42-59`, `internal/shedadapters/archive.go:67-76` and `internal/shedadapters/bouncer.go:1621-1651`.
   - The generation number lives only in the run directory's `generation-record.json`, which `CarryRecords` carries over: `internal/shedadapters/settlerecord.go:24,117-125,261-282`.
   - The record never names that file as the source of D2's "first round of a generation" or of D3's key. It also gives no rule for reading a missing record as generation 1, and Plan has to find this out.

3. **MEDIUM, design, D3 and D6 (sender of the ask; the orch-role section keyed on "mail from `<shortname>:LOOM`").**
   The ask is not always signed by the LOOM program.
   - `NotifyParent` → `postNotice` → `notifyPrime` posts as the caller's own strand whenever `agentname.StrandNameEnv` names a non-orch strand. It falls back to the program only when creel refuses the claim as coming from outside the pane: `internal/orchcli/notify.go:112,197-208`.
   - The review row runs inside the driver's `lym shed step`, so under an LLM driver strand the mail is signed by the driver strand.
   - D6's section, which triggers on `<shortname>:LOOM`, then does not match the real sender. The orch's role text trusts LOOM mail differently from a child strand's mail.

4. **MEDIUM, design, D5 ("capped so it falls before the chair's own `review_timeout_min` runs out, as `assist_reviewer_timeout_min` is capped").**
   The record states no cap formula and no outcome when the cap leaves no room, so the claim that the chair outlives the wait is not established.
   - The assistant's cap subtracts a start allowance and one await cap from the chair's timeout. It defers to `run_timeout_min` when the key is 0, and it returns a zero bound with a skip reason when no room is left: `internal/loomengine/review.go:64-84`.
   - D5's deadline is measured from the ask, which is sent before the table starts. It must also leave the chair time to consolidate after the await returns.
   - With a small `review_timeout_min` the record gives neither the deadline nor whether the round asks at all. The D5 criterion at `decision-record.md:162` covers only the ordinary capped case.

5. **LOW, consistency, D3 ("no live orch resolves … sets the state `no-parent` at once").**
   With no live orch, `NotifyParent` still resolves the prime's orch and posts: `internal/orchcli/notify.go:41-49` and `:193-195`.
   - It returns `posted=false` only when no orch strand is recorded, or as an error when the post fails.
   - A run whose parent orch is dead therefore leaves the ask `open` until it ends in `timeout`, not `no-parent`.
   - The D3 criterion at `decision-record.md:157` ("with no parent orch to reach, the ask is set `no-parent`") assumes a state the mechanism does not produce in that case.

Tool calls:
- `grep StrandNameEnv` under `internal/`: to check whether lym callers carry the strand env that decides `NotifyParent`'s signer. The manifest held `notify.go` but no caller-side evidence of the strand env.
