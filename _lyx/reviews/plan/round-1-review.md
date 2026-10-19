---
verdict: APPROVED
findings:
  - id: card7-attach-preflight-docs-stale
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/07-reedcli-attach-floor.md (internal/reedcli/attach.go:13, :40-41, :58)
    summary: Card 7 adds an envelope-aborting floor step ahead of EnsureSession but leaves three attach.go texts saying EnsureSession and Status are the only fallible pre-flight steps, one of them the operator-facing Long help
    fix: In card 7's Intent add a bullet saying the same commit rewrites the three texts to name the floor check as the first pre-flight step, ahead of EnsureSession and Status. The texts are the attach.go file header sentence at line 13 (EnsureSession and Status remain the only two pre-flight steps that can abort with an envelope error), the Long help at lines 40-41 (Every fallible step (booting the session if it is cold, then checking that the server/session is up) runs pre-flight) and the RunE comment at line 58 (Pre-flight, in a load-bearing order). The Long help and the RunE comment lie inside the attachCmd member card 7 already targets, and the header follows the plan's convention of editing a file header under a member target of that file, as cards 4 and 8 do, so no target changes.
  - id: card4-gitrepo-doc-leaf-claim
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/04-gitexec-floor-check.md (internal/gitrepo/doc.go:27)
    summary: After card 4 gitexec imports internal/logger and internal/toolfloor, but gitrepo's doc.go still says gitexec stays a zero-dependency leaf, and no card targets that sentence
    fix: Add `internal/gitrepo/doc.go` to card 4's Edit group. Add an Intent bullet saying line 27's clause "gitexec itself stays a zero-dependency leaf" becomes a statement that gitexec stays a low-level package gitrepo builds on, importing only `internal/logger`, `internal/toolfloor` and its process helpers and no gitrepo or feature package. Card 3 also edits that file, but it runs first and changes only the CherryPick bullet. A combined `validate-plan --overlay` run with this target added reported no new blocking finding against the `{}` baseline.
  - id: card3-bolt-empty-subtest-redundant
    severity: LOW
    class: design
    origin: handler
    location: _lyx/plan/03-cherrypick-without-empty.md (internal/fabricengine/boltpullfirst_integration_test.go:302)
    summary: The new Bolt subtest re-pins dropSeedCommits' failed-replay branch, which the existing subtest "a replay conflict restores the original tip" covers. CherryPick's empty-commit refusal is pinned by card 3's new TestCherryPick step, so the subtest and its new export add no behaviour
    fix: In card 3 drop the planned subtest and its export. Remove `internal/fabricengine_test#TestBolt_PullThenCommitWritten` from the Edit group, the whole `**Create:**` group with `plan:internal/fabricengine#ErrDropReplayFailedForTest`, and the Uses entries only that subtest needed (`DropSeedCommitsForTest`, `BoardDropRequestForTest`, `newBoltFixture`, `boltFixture.seedCommitLocal`, `boltFixture.moveUpstream`, `boltFixture.head`). Replace the two Intent bullets on the export and the subtest with one sentence. It says dropSeedCommits' handling of a failed replay, abort, restore of the start tip and errDropReplayFailed, takes the same branch for every CherryPick error and is pinned by `TestBolt_PullThenCommitWritten`'s "a replay conflict restores the original tip". With TestCherryPick's empty-from-start step, that test meets the acceptance criterion. A combined `validate-plan --overlay` run with these removals reported no new blocking finding against the `{}` baseline.
  - id: card8-loop-scan-trips-on-doc-prose
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/08-reed-session-row-prime-first.md (TestStatusFormats_SessionLoopsTakeOneFormat bullet)
    summary: The new scan fails on any non-test reedengine file but bar.go that contains the loop opener `#{S:`, yet the same card has internal/reedengine/doc.go describe the new two-loop row, so writing the loop's spelling in that doc prose fails the card's own test
    fix: In card 8's `TestStatusFormats_SessionLoopsTakeOneFormat` bullet, narrow the cross-file rule. It fails only when a Go string literal (an `*ast.BasicLit` of kind STRING, found by parsing each non-test `.go` file of `internal/reedengine` other than `bar.go`) contains `#{S:`, so comments and doc prose are not scanned. The bullet keeps its stated purpose, that a session loop declared elsewhere cannot escape the scan.
  - id: card6-spawn-check-preempted-by-runner
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/06-reed-tmux-floor-check.md (internal/reedengine/lifecycle.go:457-466; internal/reedengine/lock.go:71; internal/reedengine/revive_test.go:24-26)
    summary: In production the boot runs hookless TmuxCmd commands over the same cfg.Tmux path before spawnSession, so the runner refuses a below-floor tmux from the shared cache first; only the hook-driven test row reaches the spawn-time check, and card 6 does not say so
    fix: In card 6's server-spawn bullet add a sentence keeping the call D4 names but stating what it is. In production the boot's earlier runner commands over the same `e.cfg.Tmux` path (the capability probe's `-V`, `list-commands`, `has-session`) refuse a below-floor tmux first from the same cache entry, so the spawn-time check is a backstop for a spawn the runner did not precede. The row "the server spawn refuses before running tmux" reaches it only because a hook runner never reads the process cache. No code or target change.
---

### [MEDIUM] Card 7 leaves the attach command's pre-flight texts stale

**Location:** `_lyx/plan/07-reedcli-attach-floor.md`. The stale texts are in `internal/reedcli/attach.go`: the file header at line 13, the `Long` help at lines 40-41 and the `RunE` comment at line 58.

**Issue:** Card 7 puts `reedengine.CheckTmuxFloor` at the top of the attach `RunE`, before `EnsureSession`, and its refusal is reported on the command's envelope.
Three texts in the same file still describe the pre-flight as two fallible steps, booting the session and then checking it.
The header says `c.eng.EnsureSession()` and `c.eng.Status()` "remain the only two pre-flight steps that can abort with an envelope error".
The operator-facing `Long` help lists the same two.
Card 7's Intent names none of the three, so after the card two comments and the command's help text are false.
CLAUDE.md asks that a change to observable CLI behaviour land its docs in the same commit.

### [LOW] Card 4 leaves gitrepo's "zero-dependency leaf" sentence about gitexec

**Location:** `_lyx/plan/04-gitexec-floor-check.md`; the stale sentence is `internal/gitrepo/doc.go:27`.

**Issue:** Card 4 makes gitexec import `internal/logger` and `internal/toolfloor`.
`internal/gitrepo/doc.go:27` still says "gitexec itself stays a zero-dependency leaf".
The sentence was already inaccurate, since `go list` shows gitexec importing `internal/pauserecord` and `internal/proc`.
Card 4 is the change that widens the gap, and no card targets the sentence.
Card 3 edits that file only for the CherryPick bullet at line 77.

### [LOW] Card 3's new Bolt subtest repeats an existing subtest's coverage

**Location:** `_lyx/plan/03-cherrypick-without-empty.md`; the covering subtest is at `internal/fabricengine/boltpullfirst_integration_test.go:302`.

**Issue:** `dropSeedCommits` (`internal/fabricengine/destroy.go:1769-1779`) handles every non-nil `CherryPick` error in one branch: abort, `ResetKeep(startTip)`, and `errDropReplayFailed` wrapped around the replay error.
The existing subtest "a replay conflict restores the original tip" reaches that branch and asserts its outcome: the diverged skip, head back at the original tip, a clean board and no record entry.
The empty-from-start half is pinned at the gitrepo boundary by card 3's own new `TestCherryPick` step, which expects the refusal with `CHERRY_PICK_HEAD` present and `CherryPickAbort` restoring HEAD.
The planned subtest "an empty local commit fails the replay and restores the original tip" adds a different input to the same branch and no new behaviour.
It also forces the new `ErrDropReplayFailedForTest` export, which exists only for that subtest.
`PATTERN-test-economy` and the rubric's test-economy bullet rule it out.

### [LOW] Card 8's loop scan fails on the doc prose the card itself writes

**Location:** the `TestStatusFormats_SessionLoopsTakeOneFormat` bullet of `_lyx/plan/08-reed-session-row-prime-first.md`.

**Issue:** The planned test fails "when any non-test `.go` file of `internal/reedengine` other than `bar.go` contains `#{S:`".
The same card asks `internal/reedengine/doc.go`'s geometry-pin paragraph (around line 636) to describe the new row.
That row is two single-format `#{S:}` loops, the spelling the decision record and the card use for it.
An implementer who writes that spelling in the doc comment fails the card's own gate.
`bar.go`'s header already spells `#{S:}` in prose (line 2), which shows how natural the spelling is in a comment.
The rule's purpose is to catch a session loop declared outside `bar.go`, and such a loop is always a Go string literal.

### [LOW] Card 6 does not say that the runner pre-empts the spawn-time floor check

**Location:** `_lyx/plan/06-reed-tmux-floor-check.md`, the server-spawn bullet. Evidence: `internal/reedengine/lifecycle.go:457-466`, `internal/reedengine/lock.go:71` and `internal/reedengine/revive_test.go:24-26`.

**Issue:** The engine's runner is `NewTmuxCmd(cfg.Tmux, …)` (`lock.go:71`).
On the boot path, hookless runner commands go out before `spawnSession` runs, which is why `reviveTestEngine` has to script `-V`, `list-commands` and `has-session`.
Each of those passes card 6's `checkFloor`, which calls `CheckTmuxFloor(p.tmuxPath)` over the same path and cache entry that the spawn's `CheckTmuxFloor(e.cfg.Tmux)` reads.
So a below-floor tmux is refused by the runner first, and in production the spawn-time check never fires as the refusal.
The planned row reaches it only because a hook-driven runner never reads the process cache.
D4 names the spawn site, so the call stays.
The Intent should say it is a backstop, so that neither a reader nor a later change takes it for the boot's first refusal.

## Rejected

- guard-reach: `tools/deploy` spawns git outside gitexec with no stated bound. Rejected: D4's floor check covers the commands `lyx` runs, and `tools/deploy` is a separate program. The Land row runs it after a `lyx` process whose own git commands already passed gitexec's check on the same host. The D5 audit covers the git features `tools/` uses, and card 2 records them.
- way-forward: a cached below-floor refusal outlives an upgrade in a long-lived process, and its "install git … or newer" step needs a restart there. Rejected: card 4's bound is D4's own ("a long-lived daemon keeps its parsed result and is not rechecked after a git or tmux upgrade during its life"), which covers a cached refusal as well as a cached admit. The decision record settled it.
- way-forward: card 5's floor error reaches `postBatchChecks` without naming a webster resume step, against `PATTERN-refusal-way-forward`. Rejected: card 5 adds no webster refusal of its own. The error is the toolchain floor error propagated from `Delta`, and it names its way forward. In the same process, gitexec's exec-core check raises it at an earlier git command, as card 5 states.
- Untested guard calls in cards 5 and 7 (`planglyph.Delta`, `reedcli` attach), weighed under the rubric's refusal-reachability bullet. Not raised: the decision record's acceptance criteria say these two packages cannot reach the process caches from their tests, and that the calls are held by review. Raising it would overturn that decision.

## Focus answers

- Cards 4, 6 and 7: card 4 and card 6 each state the unparseable-line admit (one Warn, then cached as admitted) and the per-process cache. Each bounds it: only a parsed below-floor version is refused, nothing bypasses a parsed refusal, and a long-lived process keeps its answer. Card 6 also bounds the hook bypass and the Windows skip. Card 7 adds a guard and introduces no skip of its own. No defect.
- Card 3: every prescribed branch names a reaching input. The empty-from-start refusal is reached by the `commit --allow-empty` step. The in-progress probe's exit 1 is reached by the nonexistent-SHA step. The index-differs return is reached by the failing `prepare-commit-msg` hook step, and gitkit strips only `*.sample` hooks, so the hook runs. The failing-probe, `diff-tree` and `--skip` returns are declared fail-loud guards. The existing Bolt conflict subtest does cover the planned Bolt subtest; see `card3-bolt-empty-subtest-redundant`.
- Cards 9 and 10: `PausedCalls`, `awaitMarkedProbe`, the lock-wait seam and its dispatch duplicate no existing helper or test. Neither card uses a seam or helper that a later card creates.
- Card 8: no Edit member changes its parameters, results or receiver, so no re-sign arrow is missing. The rendered-row check stays in the `tmux && linux` file `bar_integration_test.go`.
- Card order and verify: no card uses a later card's target. Every targeted package is run by the untagged and integration tiers, and the reed packages also by the tmux tier. No card targets `internal/gitkit` or `tools/`. The doc-only targets under `docs/` are guarded by `TestEnforcement_MarkdownLinks`, a `//lyx:guard` test every card gate runs.
- Fidelity: D1 is carried by card 1, D2 by card 3, D3 by card 8, D4 by cards 2 and 4–7, D5 by card 2 with cards 3 and 8, D6 by card 9 and D7 by card 10. The two allow-set-title and floor entries are carried by card 2. Every Constraint is carried. No card edits this task's own board entry.
