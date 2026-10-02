---
round: 2
focus:
  - "Delivery bookkeeping across restarts and processes, in `_lyx/discussion/decision-record.md` D3, D4, D5 and D6. D5 has the `parent-review` closure re-send the delivery prompt 'a minute after the last prompt', up to three prompts per round, and states the bound 'the parent receives at most three notices per round from the gate, plus one per notify run'. D6 has `lyx loom review notify`, run from the hub's prime worktree in a separate process, make the gate send one more prompt at the writer's next turn boundary. D4 says the closure decides from the current round's directory, for idempotence across an attach, and D3 lists that directory as the request, the delivery record, the verdict and the review file. Where do the prompt count, the last-prompt time and a pending notify live? Does D5's stated bound still hold after a driver restart? How does notify reach the gate? Does a prompt whose send fails (D2 leaves the entry pending) count toward the cap?"
---

All seven round-1 findings are fixed.
This entry covers the one gap the judge pass found.
D5's notice bound and D6's `notify` rely on per-round state that D3 never places on disk.
