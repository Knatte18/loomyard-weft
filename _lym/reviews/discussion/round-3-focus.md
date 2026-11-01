---
round: 3
focus:
  - "Runner liveness on every platform: in D1's process file and join rule, check that the start reading the join compares tells a live runner from a reused pid on Linux (proc.StartTime) and on Windows (proc.StartTime returns an empty reading there; proc.CreationTime exists), and that D3's Windows bound covers the join as well as the end. Settle whether a Windows front can join a live runner."
  - "Cap accounting: in D1 and D4, a starting front waits on the key lock, then up to 30 s for its runner's claim, then on the exit file. Settle whether the resolved cap bounds the agent's whole wait from the front's start, so that a cap D4 accepts (below the role's lifetime) cannot return the agent to a cold cache; the declared PATTERN-keep-warm-cap makes the resolved cap the bound of the agent's wait."
  - "Card-verify runner death: in D3 and D6, a card-verify runner runs its line through verifyrun.Run, which sets the shell's process group with no parent-death signal (internal/verifyrun/run.go), while lym gate test tiers start through proc.StartTied. Settle what happens to the line, and to the slots a --wait gate child holds, when the runner is killed without an exit file, and whether D3's claim that the kill frees the slots holds for both runner kinds."
  - "Abandoned mark coverage: in D5, the build step's record is written in internal/gatecli/build.go and a blocked tier's record in reportBlocked, outside execution.report. Settle whether every record of a cancelled call carries the mark, and whether lym gate report treats marked records the same way as the median does."
  - "Lease ordering at start failure: in D1, the runner checks the lease, then creates its process file, while the front may expire the lease at its 30 s start failure. Settle whether a claim made after that expiry is refused, and whether D3 names every write that moves the lease earlier."
---

Round 2's one BLOCKING finding and two MEDIUM design findings drive the first three entries.
The fourth follows the reopened median key; the fifth covers the late-claim race and the lease-write wording.
