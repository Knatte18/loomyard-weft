---
round: 1
focus:
  - "internal/loomcli/wiring.go, start.go, sharedbootstrap.go, landingdeps.go and internal/battencli/wire.go: both sides edited the wiring. Do the pause-reader, sleep-clock and refusal seams the task added still get filled in every wiring path after the parent's new alignment, NoticeLog and peers seams, and does each drift guard and seam test still cover both sets?"
  - "internal/battenshed/notice.go, deps.go and internal/battencli/cli.go: the parent added NoticeLog to InnerRunDeps while the task changed batten's pause and idle reads. Does a notice composed on a pause-covered path still log once, and is the resume-line ordering the task's test pins still true with the parent's notice-log change?"
  - "internal/websterengine/beginbatch.go, state.go, doc.go: the parent changed the kick-start pack and EditedDoneCards, and the task removed the webster pause flag and gate. Does begin-batch still run its gates in the order doc.go states, with no reference left to the removed pause flag, and does state.go still read old state files that carry the retired pause field?"
  - "pattern/PATTERN-daemon-wakeups.md (the parent added the statuspane loop to its list of backed-off loops): does any waker or poll loop the task added, in the sleep detection, the watchdog or the orch watcher, follow that pattern's backoff rule, and does the parent's statuspane loop need the task's awake-time or pause-covered handling?"
  - "contracts/specs/refusal-spec.md, contracts/stencils/claims_test.go, contracts/stencils/orch/orch-template-role.md: both sides edited the refusal and orch-role text. Do the task's pause and resume refusals and orch instructions still match what the parent's alignment-agent landing and orch text now say, and does claims_test still pin them?"
  - "cmd/lyx/daemonwakeups_test.go and cmd/lyx/testdata/test-budget.yaml: after both sides added tests, is any daemon-wakeups scan or budget row stale or double-counted, and do the merged packages stay within their rows?"
  - "docs/overview.md and the doc.go files of battenshed, orchengine, reedengine, websterengine: did the merge leave the module table, the execution stack or a doc.go describing both the removed pause behavior and the parent's new behavior?"
  - "internal/testkit/envkit/envkit.go: the parent added helpers while the task changed the environment the tests read. Does any new parent test or helper depend on an environment variable the task's pause or sleep tests now set or clear?"
---

The orch-pause, reed-resume-hold and landing paths are where both sides edited the same wiring and docs.
Defects confined to one side's commits are out of scope.
