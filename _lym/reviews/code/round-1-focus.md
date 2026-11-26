---
round: 1
focus:
  - "internal/websterengine/verifygate.go pass() and auditledger.go reconcileAcceptHead: which commits in lastBatchHead..FixedHead did the gate's commit check vet? Trace commits that land below the pre-fix head or move HEAD without a fixer, and settle whether accept-audit can tolerate any of them."
  - "internal/websterengine/verifygate.go: preFix lives in the gate closure while PreFixHead and FixedHead persist. Does a verify-fixer commit still clear accept-audit when the run restarts between the failed and the passing evaluation, when the gate ends in a terminal failure, and after several evaluations or a flaky rerun? Compare with the amended entry's third Idea bullet."
  - "FixedHead lifetime: check every path that rebaselines, archives or restarts a run (reset targets, --fresh, auto-rebaseline, recover-batch) for a stale FixedHead that still satisfies the ancestry check."
  - "internal/websterengine/findingtext.go recordsSideStatus: can a path that changed during the run read as clean? Check records-side syncs that commit before accept-audit runs, which _lym content outside the webster, reports and plan directories is run state or evidence, untracked files, deletions of tracked files, directories and symlinks, and whether the git seam's IgnoredPath and PathDiffers behave as assumed when run inside the records directory."
  - "internal/websterengine/auditledger.go AcceptPendingAudit and runlevel.go pendingPathsWayForward: the first splits only the unverifiable paths, the second splits all of rest. Do both classify every path the same way, including a standalone layout where _lym is inside the task worktree, relative versus absolute spellings, and contract or plan paths?"
  - "internal/websterengine/audit.go CheckParent and runlevel.go Run: is State.MasterNotePath set and persisted for every Master whose transcript either audit site reads, including a resumed or re-attached run? Is the path matched in every spelling a transcript can record, and is the exemption no wider than that one file?"
  - "Test coverage against the mutation report: settle whether the saved MasterNotePath in Run, the records-side step in the pending way forward, recordGatePass's change flag and the recordsSideStatus branches have checks, and whether reconcileAcceptHead's refusing cases (FixedHead not above the last batch head, a clean parent merge above FixedHead) are exercised."
  - "Wording: read the Master stencil's 'What you never do' section, including the following 'you write only your two contract files' sentence, the acceptaudit.go long help, the doc comments and _lym/landing/summary.md (its Merge danger door and blast radius too) against what the code does."
---

Seed focus for round 1, drawn from the amended board entry and the non-test diff of the review range.
The entries name questions to settle; none is a verified finding.
