---
verdict: BLOCKING
rationale: "All seven round-1 findings are fixed in the pair, but D5's three-prompt cap and once-a-minute throttle and D6's notify verb depend on per-round state (prompts sent, last-prompt time, pending notify) that D3's directory list omits, so the stated bound on parent notices does not survive a driver restart and notify has no stated channel to the gate"
---

## Round-1 findings, checked against the edited pair

The report has no `## Focus departures` section, so there is nothing to ratify or reject.

- **F1 (BLOCKING), mid-turn sends.** Resolved.
  D2 now says the closure never sends and a pending result carries the text.
  It says every gate send and every tick-time re-evaluation happens only at a turn boundary, defined as the last arrival being Done with nothing sent since.
  It makes a delivery turn's Done arrival an ordinary arrival, and a failed delivery-prompt send a Warn that leaves the entry pending.
  D4's consumed-marker bound now covers a failed re-prompt send.
  The acceptance criteria cover a reject submitted mid-turn and a failed delivery-prompt send.
- **F2 (MEDIUM), re-send bound.** Resolved in wording: D5 caps the gate at three prompts per round, a `--failed` record does not stop the re-send, and D12 starts no second fork for a repeat notice.
  The cap's persistence is the new finding below.
- **F3 (MEDIUM), D6 attack surface.** Resolved.
  D6's Bound paragraph covers self-approve, `delivered` without a send, `reject` and `notify`, and names the Discussion-Review rounds and the committed request directory as what still sees every outcome.
- **F4 (MEDIUM), writer write boundary.** Resolved.
  D8 names both carve-outs in the writer stencil's "What you may write" section, and the stencil acceptance criterion covers them.
  This matches the stencil at `_board/_lyx/stencils/loom/loom-template-discussion.md`, which today limits the writer to its two files and gives `## Review rounds` to the gate.
- **F5 (MEDIUM), reed-design residue.** Resolved.
  Q10 is resolved under SendMessage, Interview turn 4 is marked replaced, the hub-wide name-resolution test is gone, and Scope names both orch stencils.
- **F6 (LOW), two addresses.** Resolved.
  D5 has the prompt name the seed's full name and says `$LYX_PARENT` is reed's export of the same field.
- **F7 (NIT), process history.** Resolved.
  D5's rationale and the auto-mode assumption no longer carry the attribution or date; the hub review's history stays in the support log.

## New finding: the delivery bookkeeping has no stated home

D4 says the closure "reads the current round's directory" and decides from it, and its rationale is idempotence over on-disk state across an attach.
D5 makes the closure re-send "a minute after the last prompt" up to three prompts per round, and states the bound "the parent receives at most three notices per round from the gate, plus one per `notify` run".
D6 has `lyx loom review notify`, run from the prime in another process, make "the gate send one delivery prompt at the writer's next turn boundary".

D3 enumerates the directory as the request, the delivery record, the verdict and the review file.
None of these holds the prompt count, the last-prompt time, or a pending `notify`.
If the count and time live in memory, a driver restart resets them, so the attach re-sends at once and the three-prompt bound restarts; the bound D5 states is then false under the restart case D3 and D4 exist to handle.
The `notify` verb has no stated way to reach the gate at all.
The plan writer needs the state's location and the record's fields, and whether a prompt counts toward the cap when its send fails (D2 leaves the entry pending after a failed send).
