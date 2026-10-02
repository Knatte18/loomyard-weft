# Fixer report — Discussion-Review round 1

All seven findings fixed; `lyx loom validate-discussion` exits ok on the edited pair.

## Changed

- **F1 (BLOCKING), mid-turn sends.**
  D2 now says the closure never sends; a pending result carries the delivery prompt for the loop to send.
  Tick-time re-evaluation and every gate send (delivery prompt, reject re-prompt) happen only while the writer is at a turn boundary, so a verdict submitted mid-turn is read at that turn's Done arrival.
  A delivery turn's Done arrival is an ordinary arrival, and a failed delivery-prompt send is a Warn that leaves the entry pending.
  D4's bullets return pending with the prompt instead of "attempting delivery", and its consumed-marker bound now covers a failed re-prompt send too.
  D5 says the wait loop sends the prompt.
  Two acceptance criteria were added: a reject while the writer is mid-turn, and a failed delivery-prompt send.
- **F2 (MEDIUM), re-send bound.**
  D5 caps the gate at three delivery prompts per round, says a `--failed` record does not stop the re-send, and adds a count bound on notices to the parent.
  `notify` (D6) adds one prompt per run outside the cap.
  D12 says a repeat notice starts no second fork.
  The acceptance criterion and the writer-compliance open risk name the cap.
- **F3 (MEDIUM), D6 attack surface.**
  D6 gains a Bound paragraph: self-approve, stamping `delivered` without a send, `reject` and `notify` each reach no further than this one advisory review, and Discussion-Review runs after every outcome.
- **F4 (MEDIUM), writer write boundary.**
  D8 names the two carve-outs in the writer stencil's "What you may write" section: running `lyx loom review delivered`, and appending the one parent-review entry to `## Review rounds`.
  The stencil acceptance criterion covers it.
- **F5 (MEDIUM), reed-design residue.**
  Q10 is resolved under SendMessage, Interview turn 4 is marked replaced, the hub-wide name resolution unit test is gone, and Scope names both orch stencils.
- **F6 (LOW), address.**
  The delivery prompt addresses the seed's full name, and D5 notes that `$LYX_PARENT` is reed's export of the same field.
  Goal, Scope, the first acceptance criterion and the SendMessage open risk now name the parent by its seed name, not by `$LYX_PARENT`.
- **F7 (NIT), process history.**
  The hub-review attribution and date are gone from D5's rationale, and "(revised by the hub review)" is gone from the auto-mode assumption.

The support log's Question ledger gains Q11 (the turn-boundary sends and the cap) and Q12 (the unchecked verbs' bound) as this round's auto-picks.

## Deferred

None.
