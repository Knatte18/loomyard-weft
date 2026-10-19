# Support log — ubuntu-2404-support

## Interview

### Turn 1 — exploration

Autonomous mode, no advisors.
Read the board entry, the related entries (`run-machine-ownership`, `open-window-per-machine`, `xdr-git-test-file-burst`, `doctor`, `pause-background-work`), then the code of each part: lyxtemp's run directory, tmuxkit's socket check, `gitrepo.CherryPick` and `dropSeedCommits`, the merge recheck and its integration tests, reed's `bar.go`, and creel's worker loop and the failing deliver subtest.
Local toolchain: git 2.53.0, tmux 3.6.

### Turn 2 — ly:orch: operator decision on #744

ly:orch relayed the operator's decision "Diagnose #744, never defer it": find why it fails on the host and fix that cause.
Answered that it reproduces here as a rare flake and is being diagnosed; recorded as D6.

### Turn 3 — stress runs

- `lyx gate test ./internal/creelengine -- -run 'TestScheduler_Reminders/HoldMutes' -count=300 -cpu=1,4`: 1 failure, then 2 with `-cpu=4`, the same line as the host's.
  Reading `run`, `probe` and the test's `probeWaits` gave the stale-wake race of D6.
- `lyx gate test --tags integration ./internal/fabricengine -- -run 'ForeignStateAppearingWhileWaitingForLock' -count=40`: green.
  Reading `launchBlockedOnLock` and the guard order gave the hypothesis of D7.

### Turn 4 — decisions

Settled D1 to D8 as auto-picks, listed in the question ledger.

### Turn 5 — lens review

Every fork reported.
- pattern-fit BLOCKING, the new `PATTERN-toolchain-floor` entry would cross the overview headroom (PATTERN.md 15563 bytes, line 15564): fixed, no entry added; the floor rule lives in `internal/toolfloor`'s `doc.go`, named in Constraints and Open risks.
- pattern-fit MEDIUM, D2's probes are not existing reads: fixed, they are CLI calls through `runChecked` like `MergeHeadPresent`.
- pattern-fit LOW, gitexec's probe and spawn logging: fixed, it falls under gitexec's recorded reason.
- pattern-fit LOW, no leaf enforcement test for toolfloor: fixed, added to D4 and acceptance.
- pattern-fit LOW and leanness LOW, the `PATTERN-wait-mark-display-only` bullet stretches the entry: fixed, cut; D3 cites `bar.go`'s header.
- pattern-fit LOW, premises MEDIUM (two findings) and scenarios LOW, tmux spawns outside `TmuxCmd`, the exec hook, and a per-path check: fixed in D4, the check covers the runner, the server spawn and attach, goes through the hook in tests, and runs per resolved tmux path.
- pattern-fit LOW, floor tests must not substitute a binary: fixed, version line injected.
- premises MEDIUM and scenarios LOW, an empty `NameSlug` also marks a standalone session: fixed, the marker requires a hub geometry.
- premises LOW and scenarios LOW, D2's index condition and the survival of working-tree changes: kept the condition, added the survival bound and acceptance, and `--skip` failure handling.
- premises LOW, D7's exact dirt source: fixed, it names the rewrite of a tracked file or the branch switch.
- premises LOW, D6 holds: no change.
- scenarios MEDIUM, a package-level seam is shared by parallel tests: fixed, the signal is tied to one lock path.
- scenarios MEDIUM, every verb the helper launches must reach the seam: fixed, `MergeIn`, `Merge` and `MergeAbort` named.
- scenarios MEDIUM, caching a transient probe failure: fixed, only a parsed result is cached, and the daemon-upgrade bound is stated.
- scenarios MEDIUM, the floor error read as a `*GitError`: fixed, it is its own type, with acceptance on `RunGit`.
- scenarios MEDIUM, negative checks pass vacuously: fixed, D6 syncs on a probe whose snapshot followed the post, for positive and negative checks.
- scenarios MEDIUM, an existing prime session stays unmarked: fixed, the option is set on boot too, and the empty state is stated.
- scenarios LOW, no collision-retry scenario: fixed, added to acceptance.
- goal-scope MEDIUM, the audit has no acceptance criterion and a vague record rule: fixed, D5 lists features after git 2.20 or tmux 3.0, with acceptance.
- goal-scope MEDIUM and leanness MEDIUM, D7 states an unconfirmed cause as fact: fixed, D7 names a slowed reproduction that confirms it, with git 2.43 as the open cause otherwise.
- goal-scope LOW, D4 departs from "at start-up" silently: fixed, D4 says it replaces that wording.
- goal-scope LOW, unparseable git admitted silently: fixed, stated in D4's bullet.
- goal-scope LOW and leanness LOW, conditional docs/overview.md bullet: fixed, flat; the module tree lists leaf packages.
- leanness MEDIUM, D8 duplicates acceptance: fixed, cut, its one requirement moved to Constraints.
- leanness LOW, D6 evidence numbers and D1 provenance duplicated: fixed, the numbers stay in Turn 3, provenance in the notes.
- leanness LOW, D1 and D4 rationales re-argue rejected alternatives: fixed, cut.
- leanness LOW, auto-mode assumptions restate D6 and D7: fixed, trimmed.
- leanness MEDIUM, `--empty` criterion too broad: fixed, limited to `cherry-pick` argv.
- leanness LOW, `sweepStep` is not part of the race: fixed in the notes.
- board-fit LOW, XDR as the slowness source: fixed, named in D7 and Scope.
- board-fit LOW, #739 must land first: fixed in D1.
- board-fit LOW (three), `reed-status-buttons`, `fabric-monorepo-verbs`, `orch-any-worktree` and `creel-agents` context: rejected, context only, no conflict.

## Rejected alternatives

- #739: give tmuxkit a shorter socket directory than the run directory. Rejected: a second temp location beside lyxtemp's run directory, against `PATTERN-lyx-temp-root`, while the short name fixes the arithmetic for any uid.
- #743: detect the git version and pass `--empty=drop` only where supported. Rejected: two code paths with different behaviour; the operator chose one path for every version.
- #743: replay with `git am` or `rebase --onto` instead of cherry-pick. Rejected: a larger change to `dropSeedCommits` for no behaviour gain.
- #738: detect the tmux version and keep the two-format loop on newer tmux. Rejected: two formats to test; the single-format loop runs on both.
- #740: identify the prime by comparing the session name with the hub's short name. Rejected: ties the format to hub naming; a session option is set where the geometry already knows it is the prime.
- Floor check in the root command's pre-run. Rejected: every command would spawn git and tmux, including ones that need neither, and a machine without tmux would fail commands that never touch it.
- Floor below the host's versions, set by the audit's highest feature. Rejected: no test or host proves those older versions.
- A config key to skip the floor check. Rejected: YAGNI; nobody asked, and an unparseable version is already admitted.
- #744: change the scheduler to report each probe's message set to the test. Rejected: production code for a test's synchronization, when the test can wait on the outcome it asserts.
- Add `PATTERN-toolchain-floor` and cut the same bytes from other entries. Rejected: rewording other modules' entries is outside this task, and the overview sits one byte under its headroom line.
- D7: lengthen the 150 ms wait. Rejected: still a guess at the state instead of a check of it.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 — Which #739 fix? Resolved (auto-pick): the six-hex run name from #745, since it is host-tested and stays in lyxtemp.
- Q2 — How does CherryPick tell an empty pick from a conflict? Resolved (auto-pick): pick in progress, no unmerged entries, index equals HEAD; only then `--skip`.
- Q3 — What sets the floor? Resolved (auto-pick): the host's git 2.43.0 and tmux 3.4.
- Q4 — Where does the floor check run? Resolved (auto-pick): once per process in gitexec and in reed, off Windows for tmux.
- Q5 — What does an unparseable version do? Resolved (auto-pick): admitted, logged by reed.
- Q6 — How is the prime's session identified? Resolved (auto-pick): a session user option set at prime session creation.
- Q7 — Where do the audit results live? Resolved (auto-pick): `internal/toolfloor`'s `doc.go`.
- Q8 — Is #744 a host fault? Resolved (operator decision via ly:orch to diagnose; cause auto-picked from the stress reproduction): a race in the test's synchronization.
- Q9 — Is the merge-lock failure git 2.43? Resolved (auto-pick, unconfirmed here): no, the fixed 150 ms wait; see Open risks.
- Q10 — How do tests reproduce git 2.43 and tmux 3.4? Resolved (auto-pick): through properties of lyx's own code, D8; the operator's host run after landing covers the real tools.
