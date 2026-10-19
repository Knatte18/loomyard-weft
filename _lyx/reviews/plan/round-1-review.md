---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/09-creel-deliver-marked-probe.md:35
    summary: awaitMarkedProbe advances the fake clock as soon as it sees the marked pause read, which happens before the worker registers its wait, so the advance can be lost and the read at mark+2 never comes, a new race in the fix for #744
    fix: "In card 9's awaitMarkedProbe bullet, replace advance-then-wait with a sequence that cannot lose the advance, and say why it cannot. For example: after awaitPauseRead(t, receiver, mark+1), loop under the real-time deadline until PausedCalls(receiver) reaches mark+2. Each pass waits for the fake clock to record one more probe wait than it held when the marked read was seen (e.clock.AwaitWaitsWhere with isProbeWait), then advances by the 8 s ceiling. A pass whose advance landed before the worker parked is retried. Then state in the same bullet the bound on the total time advanced against the holds, TTLs and back-offs the negative checks run under, in place of the single-8-s claim. A creelfake addition that gives the helper a per-receiver parked signal is an equally valid fix; whichever is chosen, the bullet must name the state it waits for before Advance runs."
  - id: F2
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lyx/plan/03-cherrypick-without-empty.md:47
    summary: Card 3 prescribes and pins an error for an already-carried root commit, but git --empty=drop drops one with exit 0 (confirmed on this host's git 2.53), so the pick result changes on newer git against D2 and the same-pick-result constraint
    fix: "In card 3's Intent, replace the rev-parse comparison of <sha>^{tree} with <sha>^1^{tree} by `git diff-tree --quiet --root <sha>` through runChecked. Exit 0 means the commit introduces no change against its first parent, or against the empty tree for a parentless commit, and so is empty from the start. Exit 1 means it has a change, and the card then runs --skip. Any other exit is the wrapped git error, a fail-loud guard. Reshape the TestCherryPick step to an already-carried root commit that returns nil, with HEAD unmoved and no CHERRY_PICK_HEAD, matching --empty=drop. Drop the claim that the comparison-error exit is the one reachable exit."
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/plan/00-overview.md:43
    summary: Card 5 targets internal/websterengine#postBatchChecks, but neither the untagged nor the integration verify line runs ./internal/websterengine, whose tests include 14 integration-tagged files
    fix: In 00-overview.md's `## verify:`, add `./internal/websterengine` to both the untagged `lyx gate test` line and the `--tags integration` line, after `./internal/testkit/creelfake` and before `./cmd/lyx`. validate-plan --overlay admits it.
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lyx/plan/08-reed-session-row-prime-first.md:50
    summary: Card 8's unmarked render keeps TestBar_LineTwoListsSessionsInIDOrder's frame, whose existing assertion puts the client's session before a-very-long-session-name, but tmux 3.4's S loop orders by name and the fixture's session is named worktree, so the row fails on the floor host
    fix: "In card 8's TestBar_LineTwoListsSessionsInIDOrder bullet, state that the unmarked render drops the existing current-before-other order assertion, since the S loop's order is session name on tmux 3.4 and session id on newer tmux (D3). That render asserts only once-each, lit with the full name, and no comma. The prime-first order is asserted only by the two marked renders, and the follow-button half keeps its last-session-button anchor. Also say the test's comment no longer claims session-id order."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/04-gitexec-floor-check.md:34
    summary: After card 4 the floor error comes back unwrapped and is no *GitError, yet gitexec's file header and the godocs of runCore, Run, RunStdin and RunGit still define such an error as git could not be run at all, and card 4 names none of these comment changes
    fix: "In card 4, move `internal/gitexec#Run` and `internal/gitexec#RunGit` from Uses to Edit, and add `internal/gitexec#RunStdin` to Edit (the combined validate-plan --overlay run admits this). Extend the runCore bullet: runCore's godoc, the godocs of Run, RunStdin and RunGit, and gitexec.go's file header comment name the floor error as the other non-*GitError result, which no git command ran to produce. Name the file header in the Intent only, since listing gitexec.go beside its members is refused by redundant-file-target."
  - id: F6
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: _lyx/plan/03-cherrypick-without-empty.md:49
    summary: Card 3 rewords errDropReplayFailed because an empty-from-start commit now fails the replay too, but leaves the same replay-conflicts claim in Bolt.PushRecorded's godoc (bolt.go:80) and in fabricengine/doc.go:808 and :814
    fix: "Add `internal/fabricengine#Bolt.PushRecorded` to card 3's Edit list (the combined validate-plan --overlay run admits it). Extend the errDropReplayFailed bullet: PushRecorded's godoc and fabricengine/doc.go's two Bolt paragraphs, the skip list and the Push retry paragraph, say the replay failed instead of conflicted. doc.go is already a card target."
  - id: F7
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: _lyx/plan/02-toolfloor-package.md:40
    summary: Card 2 adds the new stdlib-only shared leaf to docs/overview.md's module tree only, while docs/shared-libs/README.md lists every implementation-only shared library, internal/dotgit and internal/gitexec among them
    fix: "Add `docs/shared-libs/README.md` to card 2's Edit list (the combined validate-plan --overlay run admits it). Add a bullet: its Implementation-only libraries list gains `internal/toolfloor` — the git and tmux version floor and its version-line parser, a stdlib-only leaf. Place it beside `internal/dotgit`."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/07-reedcli-attach-floor.md:10
    summary: "Card 7 justifies the attach check by the pre-flight's degrade to the bare argv, but attach's RunE first calls EnsureSession and Status, which run tmux through the same TmuxCmd runner and cached check on the same cfg.Tmux path and refuse first, so the added call as placed never produces the refusal"
    fix: In card 7's Intent, place the CheckTmuxFloor call at the top of RunE, after the ShouldAbort check and before EnsureSession, so a below-floor tmux is refused first with the floor error on the envelope, unwrapped. Replace the degrade-to-bare-argv reason with that one.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/03-cherrypick-without-empty.md:38
    summary: Card 3 calls the index-differs return unreachable by a plain pick, but a failing prepare-commit-msg hook stops the pick with exit 128, CHERRY_PICK_HEAD set, nothing unmerged and an index that differs from HEAD (confirmed on this host), so the branch is reachable and needs a reaching step
    fix: "In card 3's Intent, drop the index-differs return from the fail-loud list. Add a TestCherryPick step, \"a pick whose commit step fails returns git's error with the pick in progress\": it installs an executable prepare-commit-msg hook that exits 1 in the test repo's hooks dir, picks a commit with a change, and expects the wrapped *gitexec.GitError, CHERRY_PICK_HEAD present, then CherryPickAbort nil and HEAD restored."
  - id: F10
    severity: LOW
    class: decision
    origin: handler
    location: _lyx/plan/04-gitexec-floor-check.md:28
    summary: D4 has gitexec run git --version through its own exec core, but card 4 runs the probe with exec.Command directly, never through runCore, and does not say why it departs
    fix: "In card 4, either split runCore into the floor-checked entry and an unexported spawn half, and have probeGitVersion call that spawn half, so the probe runs through the exec core as D4 says; or keep the direct exec.Command and add one sentence to the probeGitVersion bullet. That sentence says the probe bypasses runCore only because runCore's first step is the floor check itself, and keeps runCore's local-command environment (childEnv(nil), proc.HideWindow)."
  - id: F11
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/01-lyxtemp-short-run-names.md:30
    summary: Card 1 asserts the new run directory's mode is 0700 unconditionally, but Windows reports no Unix permission bits, and the package's own root_test.go checks the root's 0700 mode only when runtime.GOOS is not windows
    fix: In card 1's TestNewRun_HoldsItsLockUntilRemoved bullet, assert the 0700 mode only off Windows, as root_test.go does for the root, and keep the name-pattern assertion on every platform.
  - id: F12
    severity: LOW
    class: design
    origin: handler
    location: _lyx/plan/06-reed-tmux-floor-check.md:38
    summary: Card 6 lets a TmuxCmd with an exec hook skip the process floor cache, admitting when floorLine is empty, but its Intent states only that hook-driven tests run no probe, not what bounds the bypass in production
    fix: "In card 6's floorLine and checkFloor bullet, add the bound: execHook and floorLine are set only by reedengine's own _test.go files, so no production TmuxCmd carries a hook, and every production run, runStdin and output reaches CheckTmuxFloor."
  - id: F13
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/plan/02-toolfloor-package.md:36
    summary: Card 2 writes the audit into toolfloor's doc.go, including that no feature lyx uses is newer than the floor, while cherry-pick --empty=drop and the two-format S loop stay in the tree until cards 3 and 8 land, so the doc is false for those commits
    fix: In card 2's doc.go bullet, have the audit record the two above-floor forms as found and to be replaced, without claiming none newer than the floor. Have card 8, the last to remove one, edit internal/toolfloor/doc.go's audit sentence to the final none-newer-than-the-floor statement. Or reword card 2's audit sentence so it is true at card 2's commit.
---

### [BLOCKING] awaitMarkedProbe can lose its clock advance

**Location:** `_lyx/plan/09-creel-deliver-marked-probe.md:35`

**Issue:** The marked pause read is the `Paused` call inside `worker.locate`, at the start of a probe (`internal/creelengine/deliver.go:241-242`).
The worker registers its next wait through `clock.After(next)` only after the probe returns (`deliver.go:139-141`).
`creelfake.Clock.Advance` fires only timers already registered (`internal/testkit/creelfake/creelfake.go:307-320`), and `After` sets a timer's due time from the clock's time at the call (`creelfake.go:299`).
An advance made between the marked read and the worker's `After` is therefore lost.
The worker then sleeps on the fake clock, the read at `mark+2` never comes, and the subtest fails at its real-time deadline.
D6 says to wait "for a probe whose snapshot was taken after the post, and then for the wait that follows it"; the helper never waits for that wait.
This brings the class of race #744 fixes back into the fix, under the `-count=1000 -cpu=1,2,4` run the acceptance criterion requires.

### [MEDIUM] An already-carried root commit is refused where newer git drops it

**Location:** `_lyx/plan/03-cherrypick-without-empty.md:47`

**Issue:** On this host's git 2.53, `git cherry-pick --empty=drop <root>` of a parentless commit whose tree HEAD already carries prints `dropping … patch contents already upstream` and exits 0, with no `CHERRY_PICK_HEAD`.
The card's `rev-parse <sha>^{tree} <sha>^1^{tree}` fails on that commit, exit 128, so `CherryPick` returns an error, and the new step pins that error.
D2 says a pick that stops because the change is already carried is skipped and returns nil.
The constraints keep "the same pick result" on newer git.
`dropSeedCommits` never passes a root commit, since `ahead` sits over upstream, so production is not affected.
But the card prescribes a result for `Repo.CherryPick` that departs from the decision record, and tests it.

### [MEDIUM] The verify section never runs websterengine's tests

**Location:** `_lyx/plan/00-overview.md:43`

**Issue:** Card 5 targets `internal/websterengine#postBatchChecks`, rewording the comment above its `index.Delta` call.
The rubric holds every targeted package to the `## verify:` section, hermetic build-tagged tests included.
`./internal/websterengine` is missing from both `lyx gate test` lines, and the package holds 14 `//go:build integration` test files.

### [MEDIUM] The unmarked render would keep a session-id order assertion that fails on tmux 3.4

**Location:** `_lyx/plan/08-reed-session-row-prime-first.md:50`

**Issue:** `TestBar_LineTwoListsSessionsInIDOrder` asserts that the client's session comes before `a-very-long-session-name` in line 2 (`internal/reedengine/bar_integration_test.go:218-219`).
The fixture's session is `filepath.Base` of the worktree dir, `worktree` (`mouse_boot_integration_test.go:34`, `server.go:105-107`).
On tmux 3.4 the `S` loop orders by name, per D3, so `a-…` comes first and that assertion fails.
Card 8 describes the unmarked render's new checks but never says the order assertion goes.
An implementer who reshapes the test around the existing frame keeps a check that fails on the floor host, and the test's name and comment keep claiming session-id order.

### [LOW] gitexec's error-contract godocs go stale

**Location:** `_lyx/plan/04-gitexec-floor-check.md:34`

**Issue:** `gitexec.go`'s header (lines 9-14) defines a non-`*GitError` error as "git could not be run at all".
So do `runCore`'s godoc (line 119), `Run`'s godoc (lines 200-208) and, by reference to `Run`, `RunStdin`'s.
`RunGit` returns that error as its exec failure.
After card 4, the unwrapped floor error comes back in the same slot.
Card 4 targets `runCore` but never says its godoc changes, and it lists `Run` and `RunGit` only under Uses.

### [LOW] Card 3 leaves the replay-conflicts wording in the Bolt docs

**Location:** `_lyx/plan/03-cherrypick-without-empty.md:49`

**Issue:** `internal/fabricengine/bolt.go:80` says "when the drop refuses or its replay conflicts".
`internal/fabricengine/doc.go:808` lists "a replay conflict" among the `diverged` reasons, and `doc.go:814` says "its replay conflicts".
After card 3, an empty-from-start commit also fails the replay, which is why the card rewords `errDropReplayFailed`; these three sentences say the same thing and are left out.

### [LOW] The shared-libs README misses the new leaf

**Location:** `_lyx/plan/02-toolfloor-package.md:40`

**Issue:** `docs/shared-libs/README.md` lists the implementation-only shared libraries, the stdlib-only leaf `internal/dotgit` and `internal/gitexec` among them (lines 29-43).
`internal/toolfloor` is a shared leaf that gitexec and reedengine both import, and card 2 does not add it there.

### [LOW] The attach check is placed after the calls that refuse first

**Location:** `_lyx/plan/07-reedcli-attach-floor.md:10`

**Issue:** `RunE` calls `c.eng.EnsureSession()` and `c.eng.Status()` before `AttachArgv`, and both put an error on the envelope and return (`internal/reedcli/attach.go:68-85`).
Both run tmux through the engine's `TmuxCmd` on `cfg.Tmux`, the path `TmuxPath()` returns (`internal/reedengine/lock.go:89-90`), so card 6's cached check refuses there first.
The Intent's reason, that the pre-flight degrades a failed tmux call to the bare argv, holds only for `AttachArgv`.

### [LOW] The index-differs return is reachable

**Location:** `_lyx/plan/03-cherrypick-without-empty.md:38`

**Issue:** On this host's git 2.53, an executable `prepare-commit-msg` hook that exits 1 makes `git cherry-pick <sha>` exit 128.
That leaves `CHERRY_PICK_HEAD` set, `ls-files --unmerged` empty and `diff --cached --quiet HEAD` exiting 1.
That is the index-differs state the card calls unreachable by a plain pick, so the branch takes a reaching step.

### [LOW] The git probe bypasses the exec core without saying why

**Location:** `_lyx/plan/04-gitexec-floor-check.md:28`

**Issue:** D4 says gitexec "runs `git --version` through its own exec core before its first git command".
Card 4 runs it "with `exec.Command` directly, never through `runCore`".
The reason, that `runCore`'s first step would be the floor check itself, is the writer's to state or to remove by splitting the spawn half out of the check.

### [LOW] Card 1's mode assertion fails on Windows

**Location:** `_lyx/plan/01-lyxtemp-short-run-names.md:30`

**Issue:** `internal/lyxtemp/root_test.go:52` checks 0700 only when `runtime.GOOS != "windows"`, since Windows reports no Unix permission bits.
Card 1 asserts the run directory's mode is 0700 unconditionally.

### [LOW] Card 6 states no bound on the hook bypass

**Location:** `_lyx/plan/06-reed-tmux-floor-check.md:38`

**Issue:** A hook-driven `TmuxCmd` admits with no process-cache read when `floorLine` is empty.
The rubric's attack-surface rule asks the Intent to state what bounds such a bypass.
The bound is that only tests set `execHook`, per `overlay.go`'s field comment, and the card leaves it unstated.

### [NIT] The audit doc is ahead of cards 3 and 8

**Location:** `_lyx/plan/02-toolfloor-package.md:36`

**Issue:** Card 2's `doc.go` records that no feature lyx uses is newer than the floor.
`cherry-pick --empty=drop` and the two-format `#{S:}` loop remain in the tree until cards 3 and 8 land.
So the doc states something false for every commit between card 2 and card 8.

## Rejected

- way-forward, LOW, card 1's bound-spent error names no next step: `PATTERN-refusal-way-forward` binds webster, shed and loom refusals, and D1 asks only that the error name the root.
- way-forward, MEDIUM, a cached below-floor refusal outlives a tool upgrade in a long-lived daemon: D4 decides "Only a parsed result is cached for the process", with the stated bound that a long-lived daemon is not rechecked after an upgrade, and cards 4 and 6 carry it as decided.
- guard-reach, NIT, the comparison-failure step is reachable only through a parentless commit: merged into F2, which replaces that step.
