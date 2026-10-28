```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of decisions that share state, then ran each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 × D2 (count reset by a `PostToolUse` claim): held. Tool calls reset Claude Code's count too, so creel's count errs early.
- D2 × D4 (rings counted with blocks, reset by `stop_hook_active` false): broken. See F1 and F2.
- D2 × D5 (cap hands the set to the typed bell): broken. See F2.
- D4 × D5 (a ring, an unanswered ring, the 5 s grace, then the bell): held.
- D2 × D4 (merged Stop command and the waiter fire in parallel on the same Stop, including a blocked one): held. A ring needs no running turn, and the next Stop's waiter ends the older one.
- D2 × D1 (ack-after-write once the merged command buffers `hook-read`'s output): weakened. See F4.
- D1/D2 × D6 and Constraints (full delivery against the 10,000-character field cap): broken. See F3.
- D2 × D3 (subagent filter on Stop): held. Settings hooks fire `SubagentStop`, not `Stop`, in subagents.
- D2/D4/D5 × pause and hold: held. All three gate on what a claim would hand out.
- D4 × D2 (partial claim at the 8 KiB budget changes the set): held while below the cap. A backlog over the cap falls into F2.
- D5 × D6 (bell set tracking kept in memory now that the journaled `reminded` records go): held. The restart repeat is stated in the Bound.
- D5 × pre-deploy sessions (D6 removes the reminder path they relied on): held as stated under Open risks.
- D7 × D5/D4 (after an interrupt, the bell or a still-live waiter, then D2): held either way the waiter question falls.
- D7 × `Delivery.Interrupt` clearing a restored prompt × D4's idle reading: held.
- D8 × D1 budget (quotes counted by `MessageLine` length): held.
- D8 × D9/D10 (`#<id>` in the line, "using the id after #" in the rule): held.
- D9 × D10 (same reply rule in prompt and help): held.
- D1 × pre-deploy `PostToolUse` hook (old settings run the new `hook-read`): held.
- Acceptance "at creel's cap … no ring follows" × D5's cap-triggered bell: gap. See F5.

**Findings**

**F1 — MEDIUM, design. D2's count-reset rule against D4's wake.**
A woken turn's Stop carries `stop_hook_active` true; the support log's rejected alternative "Gating D2 on `stop_hook_active` alone" says so.
So for a strand that only receives mail while idle, such as an orch between posts, nothing resets the count: a delivery turn that runs tools claims no mail, and no operator prompt arrives.
Each idle delivery adds one ring and one block, so the cap of 6 is reached after three unrelated idle deliveries, possibly hours apart.
After that, every idle delivery goes through D5's typed bell.
The "consecutive continuations" bound so becomes a count across unrelated turns, and D5 turns from a fallback into a routine pane path.
Evidence: decision-record.md D2, the "Creel counts" paragraph; D4, "rings it … only when … D2's count is below its cap"; support-log.md Rejected alternatives, "a rewake turn carries it as true".

**F2 — MEDIUM, design. D2's cap check against its reset on the same Stop, with D5's bell.**
The record does not say whether a Stop with `stop_hook_active` false resets the count before or after the cap check.
After a cap-triggered bell (D5), the bell turn's Stop is false and at the cap.
If the cap check runs first, that Stop claims nothing, the bell never repeats for an unchanged set (D5), and no ring happens at the cap (D4).
The mail then sits until a tool call or a new post, and the bell's own promise, "creel hands the mail to you as the turn ends", is false.
D2's ordering needs stating: reset first, then the cap check.
Evidence: decision-record.md D2, "A `PostToolUse` claim … and a `Stop` whose `stop_hook_active` is false reset the count … At 6 the Stop claims nothing"; D5, "never repeats it for an unchanged set"; D4's bell text.

**F3 — BLOCKING, design. The Constraint "8 KiB budget keeps each field under the cap" against D1, D2 and D6's full delivery.**
`ClaimAtBoundary` always hands out the first message whatever its size (internal/creelengine/verbs.go:162 doc, and lines 188-194: no budget check while `len(reading.ids)==0`).
A body may be up to 64 KiB (internal/creelengine/pool.go:17).
So a single message over about 10,000 characters overflows `reason` and `systemMessage`, and the hook output shows a preview in its place.
Creel still journals the message read, so it is lost, against "delivered in full" (D6, Goal).
The acceptance criterion covers only "a claim at the 8 KiB budget", not one oversized first message.

**F4 — LOW, design. D2's merged command against D1's acknowledge-after-write.**
To know whether `hook-read` printed a block, D2's command must capture its output before re-printing it.
`hook-read`'s write therefore succeeds into a shell pipe, and creel acknowledges the claim before Claude Code has read anything.
If the outer print fails or the hook process dies, the mail is journaled read and never delivered.
D1 says the ack order "stays as `Store.ClaimAtBoundary` has them", without noting the new layer.
Evidence: decision-record.md D1 and D2; internal/creelcli/hookread.go `writeContext` acknowledges after `out.Write`.

**F5 — LOW, consistency. The acceptance criteria against D5's cap trigger.**
The cap criterion says only that no ring follows; no criterion covers D5's bell at the cap, or the delivery that must follow it (see F2).
Evidence: decision-record.md Acceptance criteria, the fourth and sixth bullets; D5's trigger list.

Tool calls: none; my context held everything this lens needed.
