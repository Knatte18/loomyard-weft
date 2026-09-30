---
round: 1
focus:
  - "Decisions and Constraints, Driver Choice Single-Site Invariant amendment: the record rewrites only the headline clause (\"selects a spawn and nothing else\"), but CONSTRAINTS.md's first sub-bullet also says \"no code path gates behaviour on it\". Settle whether the llm arm's strand removal and its status-sidecar delete fit the amended wording as stated, and whether the record must name that sub-bullet in the amendment too."
  - "Goal versus Open risks and Scope Out: the Goal promises that an llm run \"never\" shows the loom-status strand and shows \"exactly one driving surface\", while Open risks accept a leftover band until the next start and Scope Out leaves a stray ly-drive strand on go runs. Settle whether the Goal overstates what the Decisions deliver, and which side should change."
  - "Open risks, RemoveStrand cascade: settle whether the llm arm's removal, which runs before the driver-strand probe, can take out a live ly-drive strand that sits beneath the status strand in the strand tree. That would kill a running driver on resume, which is worse than the \"older build\" case the risk names."
  - "Decisions, step path: the record rests on two claims, that ensureStatusStrand has exactly two callers (start's RunE and loomPreStep) and that the go driver never reaches step. Settle whether both hold, and whether any other verb or daemon path (resume, up, attach, the watchdog) re-adds the status strand on an llm run, since that would defeat the Goal the same way step does."
  - "Decisions, sidecar delete on the llm arm: the support log concedes a leftover sidecar is harmless to resolveStatusStrandAction. Settle whether this decision and its acceptance criterion earn their place in decision-record.md under YAGNI, or whether they belong only as deliberation in support-log.md."
  - "Notes for the plan writer, and Decisions that name helpers and test files: settle whether any of this is the plan-time detail (exact seams, file-level enumeration) that the writer stencil told the writer not to gather, and so should be cut. Before relocating anything, check that any requirement it carries (such as rewording start's long help) also appears in Decisions, Constraints or Acceptance criteria."
  - "Acceptance criteria: settle whether each criterion can be checked by a test, in particular \"step does not read the seed's driver\" and the ordering claim \"before the driver-spawn step\"; and whether a failed reed Up on the llm arm of start is covered."
---

The artifacts are internally coherent, and the interview traces each pick to code the writer read.
Round 1 should pressure-test three things:
whether the invariant amendment covers all the invariant text it touches,
whether the Goal's "never" and "exactly one" match the risks the record accepts,
and whether the destructive removal on the llm arm is safe against a live driver strand.
The last two entries check leanness: they cover content that may belong in support-log.md or nowhere.
Under the rubric, removing that content is as valid a finding as adding something missing.
