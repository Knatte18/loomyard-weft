---
round: 1
focus:
  - "internal/fabricengine PushPairAnchored and coalescePushSides: does every path send the records side first and withhold the code side after a failed or rejected records side, including flagged and lock-wait-gave-up code sides, and is the withheld side reported with the records failure?"
  - "internal/fabricengine rejected-push takeover read: is the name set handed in by the caller compared by D2's rule, are a takeover naming this machine and a skipped hand-over mark handled, and does a rejection with no takeover entry still replay as before?"
  - "Hand-over sequence (fabricengine, pairteardown): order of EndSubjectProcesses, code-tip push, archive of local tips and local parent merge, archive tag push, notice, hand-over mark write (before local removal, under the hub's .lym via internal/lymdirs), local removal, and the in-pair caller leaving removal to the parent's dobby; are failure paths between these steps safe?"
  - "Code-tip wait and grace (ReadRun CodeTipMissing, ReturnToPreviousTransition, OriginCodeTip, owed-tip leased push): is the lease value and the re-read after push right, does a given-up tip leave origin, and does the owner's guard reset only a tip off the last transition's line (narrowed by the parent's Added-after-Discussion entry)? Is the Run-Transition trailer read by newest matching commit, skipping other Warp-SHA commits?"
  - "internal/landingshed Finalize: parent push before its transition and before the board done, owner read before the merge and before each later push, parent-push rejection over origin already holding the task's work, and a Finalize cut between merge and push pushing at its next pass."
  - "Retired detached pushes: are the statuscommit per-transition spawn, the teardown-lock respawn and respawnForeignFlags/coalesceFlagged respawn fully removed with no leftover callers or stale docs, and do the detached pushes that stay (anchor change, tender prime push) still keep records-first order and read origin for a takeover?"
  - "Owed push path: PushOwed rename from PushOwedOffline, last error carried through ReadRun, runfsm verdict and refusal, and lym beam status; backoff retry behind the gate; does tender pause/resume and batten cancel act at once when origin is unreachable and owe their push, and does the cancel output name the re-run?"
  - "Admission and teardown lock: pauserecord.Admit taken before the commit and released on every exit (landed or owed), and the wait past StatusPushLockWait turning the push owed."
  - "Synchronous Bolt writes (boardengine): run claim, landing done and owner push before the writer goes on, while display-only status writes stay on detached sync."
  - "PATTERN-test-only-code: does every new function, the takeover reader and takeoverkit writer included, have a production caller in this diff? Check the PATTERN.md and pattern/ changes against the record's declared PATTERN-run-fsm add and PATTERN-pair-remote change."
  - "Commit messages and any Pattern-Departure trailers across the range, and claims in _lym/landing/summary.md that the diff does not bear out."
  - "Test economy: the redundancy report's two contained tests (TestNewCommitStatusSeam_WritesBoardStatusBeforeRunShed, TestFinalize_RecordsTheLandingCommitOnTheFirstTry) against their covering tests' assertions; behaviors added with no coverage."
---

Seed focus drawn from the decision record's D1 and D2, its acceptance criteria and its Added-after-Discussion entries.
The range spans every card, so the entries name the high-risk seams, not a verdict.
