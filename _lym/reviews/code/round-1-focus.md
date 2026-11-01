---
round: 1
focus:
  - "internal/gatecli runner and front (D1, D3): check the join key, tree and dirty-path comparison, lease writes under the key lock, and the claim and start-failure ordering by exclusive create. Does a superseded runner's late write ever reach a newer run's caller, and does an abandoned-marked exit file always answer 75 with done: false and never replay?"
  - "Lease handling (D3): is the lease only ever moved later except by the supersede and start-failure expiry, does the runner read it at most once a second including in its slot wait, and does cancel record exit 1 with the abandoned mark? Check the signal path records 128 plus the signal number."
  - "Replay of a recorded exit 75 as exit 1 with the added stderr line (D1), and the stdout-then-stderr copy order. Is the delivered mark set only after a successful copy?"
  - "D2 admission: --wait, inherited slot, no strand and zero cap all run in-process; a no-strand call waits for slots without a cap. Does every Go-run command (cardverify.Runner, battencli runDeploy and runRowCommand) drop agentname.StrandNameEnv from its environment?"
  - "D4 resolver and validation: keep_warm_cap and keep_warm_cap_roles bounds (at least 60 s, below the role's lifetime, at most the long cap), numbered-role suffix handling, fallback to DefaultAwaitCap with the Warn-once and Debug logging, and lym shuttle await outside a worktree. Do caps reach gatecli and shuttlecli only through shuttleengine.Engine behind shuttleengines?"
  - "D5 estimate: call id and Abandoned on RunRecord set in writeRecord for every record after cancel; the median of the latest five full calls with the exclusions the record lists; wall time from WaitStart, or Start where WaitStart is zero; lym gate report counts Abandoned records apart and tolerates records without the new fields."
  - "D6 card-verify: join key uses card.ID() so 4 and 4-<slug> join one run; cardverify.Runner takes the caller's context for slot acquire and line; Linux parent-death tie and TreeRecord kill on supersede; Windows job holding. Check the verifyrun tie changes for platform-split correctness."
  - "D7 and D9 text: agent-rendered gate calls drop --wait while Go sites and a card's Verify line keep it; shellguard gate rule, implementer stencil, await help and DefaultAwaitCap doc read as the record says. Is the llm driver stencil unchanged (D8)? Is a stencil rule the record did not ask to change dropped or weakened?"
  - "Declared PATTERN change: does the diff add PATTERN-keep-warm-cap with the declared text? Also check the daemon-wakeups and test-speed pattern edits for scope the record licenses, and runproc.RoleGate registration under the task slug with Info spawn logging and a test seam instead of re-exec of os.Executable."
  - "Test economy: new tests that duplicate existing coverage, per-helper tests the public surface already covers, and added behavior with no coverage."
---

Seed focus drawn from the decision record's decisions D1 to D9 and its constraints, against the stat of the review range.
