---
round: 1
focus:
  - "The two fork briefs (burler-cluster-fan.md, burler-fix-self-check.md) against gitInvocationMutates, pushMutates and forkViolation in internal/burlerengine/cluster.go: does every action the brief permits pass the audit, and is every limit the audit enforces on a throwaway repository either stated or harmless to omit? Trace in particular `git worktree add`, a throwaway under a scratch directory inside the target's working tree, `git remote add` before a push, and the push forms behind each rule constant."
  - "The rewritten READ-ONLY sentences in both briefs against what each brief forbade at the base: is any prohibition dropped or weakened beyond what stating the throwaway allowance needs?"
  - "The way forward on the review path: for each kind of violation forkViolation can return, is the text violationWayForward words true of a cluster review round, and does the wrapped error still reach its callers with the type, sentinel and halt classification they depend on?"
  - "Rule attribution in runsMutatingGit, redirectPushes and recordRemote: does the rule named for a counted command match the call that made it count, across several redirecting calls, a single call that both names a config key and changes a remote, and a command with several counted invocations?"
  - "Doc comments and the package doc in internal/burlerengine: do all statements about which commands count, including those on declarations the diff left unedited such as mutatingGitCommand and mutatingGitCommands, still agree with the code and with each other?"
  - "TestAuditClusterRound: the way-forward suffix assertion moved and is now conditional on wantWayForward being set. Is any row that asserted it at the base now checked more loosely, and does the change description's claim about which rows are checked hold?"
  - "The change description's claims, including its Merge danger door and blast radius, against the diff and the issue's three asks."
---

Seed focus for round 1, set from the board entry, the diff and the change description.
Each entry is a question to settle, not a finding.
