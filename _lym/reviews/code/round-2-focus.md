---
round: 2
focus:
  - "internal/pairteardown/leftover.go#RemoveBenchPairs with EndSession and awaitOrPass in internal/pairteardown/teardown.go: what happens to a bench pair whose own driver holds its run lock or has live registered processes, which processes are ended and which spared, and do the doc comments, internal/pairteardown/doc.go, pattern/PATTERN-pair-remote.md and _lym/landing/summary.md state that behaviour?"
  - "internal/pairteardown/leftover.go#RemoveBenchPairs: what is logged and returned for a prunable or unresolvable bench worktree, and for a failure after earlier bench pairs were removed, and do tests pin both?"
  - "The Worktree-Teardown wiring in internal/battencli/wire.go with NewWorktreeTeardown in internal/battenshed/teardown.go: under which locks do the bench pair removal and its pushes run, what does a failure leave standing, does a resumed row retry correctly, and does a test pin that ordering and failure behaviour?"
  - "internal/fabriccli/roleguard.go, benchPairVerb and isBenchPairPath: which add, remove and push calls are admitted, including a bench-form slug that another task's run owns, a writer on another recipe or profile and an unreadable run record; what does each refusal tell the writer; and does internal/fabriccli/roleguard_test.go cover each refusal?"
  - "_lym/landing/summary.md ## Merge danger: do the door and the blast radius cover what the diff pushes to origin, removes, ends and newly admits?"
---

Each entry names an area the diff touches and the question to settle there.
