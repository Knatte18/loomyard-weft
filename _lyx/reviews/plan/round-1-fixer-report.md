# Fixer report — plan, round 1

## Changes

- `card7-attach-preflight-docs-stale` — fixed as proposed.
  Card 7's Intent now says the same commit rewrites the `attach.go` file header's closing sentence, the `Long` help's list of fallible steps and the `RunE` pre-flight-order comment, so each names the floor check as the first pre-flight step ahead of `EnsureSession` and `Status`.
  The self-check widened it (see below): the header's "only remaining pre-flight addition" clause and `docs/overview.md`'s reed row, now in card 7's Edit group.
- `card4-gitrepo-doc-leaf-claim` — fixed differently.
  `internal/gitrepo/doc.go` joined card 4's Edit group, and an Intent bullet rewrites line 27's "zero-dependency leaf" clause as a low-level package gitrepo builds on, importing no gitrepo or feature package.
  I did not take the proposed import list (`internal/logger`, `internal/toolfloor` and its process helpers).
  `go list` shows that gitexec already imports `internal/pauserecord`, which is not a process helper, so the list would be wrong from the start.
  It would also pin a list of imports that goes stale with the next import.
  The bullet notes that card 3 edits that file only in its `CherryPick` bullet.
- `card3-bolt-empty-subtest-redundant` — fixed as proposed.
  I removed `TestBolt_PullThenCommitWritten` from the Edit group, the `**Create:**` group with `ErrDropReplayFailedForTest`, and the Uses entries only that subtest needed: `DropSeedCommitsForTest`, `BoardDropRequestForTest`, `newBoltFixture`, `boltFixture.seedCommitLocal`, `boltFixture.moveUpstream` and `boltFixture.head`.
  One Intent bullet now replaces the export and subtest bullets.
  It says the single failed-replay branch is pinned by "a replay conflict restores the original tip", and that this subtest and TestCherryPick's empty-from-start step together meet the criterion.
- `card8-loop-scan-trips-on-doc-prose` — fixed as proposed.
  The cross-file rule now parses each non-test `.go` file other than `bar.go` and fails only on a Go string literal (`*ast.BasicLit` of kind `STRING`) that contains `#{S:`, so comments and doc prose are not scanned.
- `card6-spawn-check-preempted-by-runner` — fixed as proposed, with the wording corrected by the self-check.
  The server-spawn bullet now says the spawn-time check is a production backstop.
  The boot's earlier runner commands over the same `e.cfg.Tmux` path refuse first from the same cache entry: the capability probe's `-V` and `list-commands`, and the boot's own `has-session` check.
  The bullet also says the hook-driven row reaches the check only because a hook runner never reads the process cache.

## Self-check

- fix-consistency, LOW: the `attach.go` header clause "this file's only remaining pre-flight addition" goes stale. Fixed as proposed: card 7's Intent names it.
- fix-consistency, LOW: `docs/overview.md`'s reed row lists two attach pre-flight steps. Fixed as proposed: `docs/overview.md` joined card 7's Edit group, with an Intent sentence saying the floor check becomes the first step and that card 2 changes only the module tree there.
- fix-claims, LOW: card 6 credited `has-session` to the capability probe, which runs only `-V` and `list-commands` (`probe.go:68`, `:83`). Fixed as proposed: `has-session` is now named as the boot's own check.
- fix-proposal, LOW: the `attach.go` header clause again. This duplicates the fix-consistency finding and is fixed there.
- fix-affected-set, LOW: the `attach.go` header clause and `docs/overview.md:395`. Both duplicate the fix-consistency findings and are fixed there.

## Disputed

None.

## Deferred

None.
