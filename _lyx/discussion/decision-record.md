# Decision record — ubuntu-2404-support

## Goal

lyx runs, and both gate tiers (`lyx gate test ./...` and `lyx gate test --tags integration ./...`) pass, on Ubuntu 24.04 with its packaged git 2.43 and tmux 3.4 under a 10-digit uid, without dropping behaviour on newer git and tmux.
A host below the supported floor fails plainly, with an error naming the tool, the version found and the floor, instead of failing in odd places.

## Scope

In:
- #739: tmux socket path too long under a 10-digit uid.
- #743: `git cherry-pick --empty=drop` on git 2.43, the two Bolt seed-commit rebase tests that fail through it, and the merge-lock foreign-state test failure seen on the same run.
- #738, #740, #748: reed's session row on tmux 3.4: each session once, no stray comma, the prime's session leftmost.
- #744: the creel reminders hold test failure, diagnosed and fixed at its cause.
- #747: an audit of every git and tmux invocation, the fixes it finds, and a minimum-version check for git and tmux.

Out:
- Multi-machine run ownership and per-machine windows (`run-machine-ownership`, `open-window-per-machine`), per the operator.
- Cortex XDR file-burst behaviour of the git fixtures (`xdr-git-test-file-burst`); that note shares the host, not the fix.
- The pause leaving background forks running (`pause-background-work`).
- A general diagnostics command (`doctor`): the floor check here is no `lyx doctor` verb.
- Running the gates on the NORCE host: the operator does that after landing, and what still fails there becomes a follow-up, per the operator.

## Decisions

### D1 — Short run directory names in lyxtemp

A run directory is `r` plus six random hex digits, created with `os.Mkdir` mode 0700 under the root.
A taken name is retried with a fresh one up to a bounded attempt count, after which creation fails with an error naming the root.
The `r` prefix stays, so the sweep's prefix rule and run directories left by older binaries keep working.
`internal/lyxtemp`'s `doc.go` and `pattern/PATTERN-lyx-temp-root.md` describe the new name; the entry line itself is unchanged.
This card lands first, since no test package runs on the host before it.

Rationale: the longest socket path becomes 105 bytes for the widest uid (4294967295), under the 107-byte limit, and the change stays inside the one package that names run directories.

### D2 — CherryPick without `--empty`

`gitrepo.Repo.CherryPick` runs a plain `git cherry-pick <sha>`.
When the pick stops because the change is already carried, it runs `git cherry-pick --skip` and returns nil, so HEAD stays unmoved, as before and on every git version, per the operator.
The pick counts as stopped-empty only when a cherry-pick is in progress, no index entry is unmerged and the index equals HEAD.
A commit that was empty from the start also passes that test, and newer git's `--empty=drop` refuses it today, so before `--skip` `CherryPick` compares the picked commit's tree with its first parent's tree: equal trees return an error naming the empty commit, with the pick in progress for `CherryPickAbort`, so `dropSeedCommits` aborts and returns `errDropReplayFailed` as it does on git 2.53.
The four probes, the three stopped-empty conditions and the comparison of the picked commit's tree with its first parent's, are CLI calls through `runChecked`, the same shape as `MergeHeadPresent`.
They run inside `CherryPick`, which `gitrepo`'s `doc.go` already lists among the CLI-bound methods, so the tree comparison stays on the CLI too and adds no exported method to the boundary test's list.
Every other failure, a failing probe and a failing `--skip` return the wrapped error and leave the pick in progress for `CherryPickAbort`.
Bound: `--skip` runs only on a pick that became empty against HEAD, so it can drop no change and no conflict; uncommitted working-tree changes that `dropSeedCommits`' `reset --keep` kept survive it.
`fabricengine`'s `dropSeedCommits` and its `doc.go` paragraph stop naming `--empty=drop`.

Rationale: `cherry-pick --empty` arrived after git 2.43, and `cherry-pick --skip` has existed since git 2.23.
The two failing Bolt tests (`TestBolt_PullThenCommitWritten`, `TestBolt_PushRecorded_DropsSeedCommitsOnRebaseConflict`) reach the cherry-pick through `dropSeedCommits` after the expected `pull --rebase` conflict, so this same fix covers them.

### D3 — Reed's session row works on tmux 3.4 and puts the prime first

Reed marks the prime's session with a session user option.
The prime is a hub geometry with an empty `NameSlug`; a standalone geometry, which also has an empty `NameSlug`, and a pair's session are never marked.
Reed sets the option, idempotently, wherever `pinGeometryOptionsLocked` runs for a prime geometry, at boot and in the attach pre-flight, the same call that pins the new `status-format[1]`.
A boot marks only the session it spawns, and a prime session that was already up when the new binary arrived is marked at its next attach pre-flight.
Bound: until a `lyx reed attach` to the prime whose pre-flight reaches the geometry pins, such a session draws in the second loop; an attach whose pre-flight degrades to the bare argv leaves it unmarked.
`status-format[1]` becomes two single-format `#{S:}` loops, both filtered by `reedSessionFilter`: the first draws only the marked session, the second every other session in the `S` loop's own order, which is session name on tmux 3.4 and session id on newer tmux.
With no marked session the first loop draws nothing.
Each loop body picks the lit style, focus and full name when `#{session_name}` equals `#{client_session}`, and the unlit style with the 12-character cut otherwise, so no loop uses the second, current-item format.
The follow button and the time stay where they are.
The option is display only, as `bar.go`'s header says of the bar's options.

Rationale: tmux 3.4's `S:` takes one format, so the current two-format loop draws every session twice with a literal comma (#738), and `S:` has no sort option in 3.4, so a prime-first order needs the prime outside the shared loop (#740, #748).

### D4 — A toolchain floor checked once per process at each tool's first use

This replaces the board entry's "check at start-up": the check runs once per process before the first use of each tool, not in the root command.
A new leaf package, `internal/toolfloor`, importing only the standard library and pinned by a leaf enforcement test, declares the floor, git 2.43.0 and tmux 3.4.
It parses a tool's version line and returns a floor error naming the tool, the version found, the floor and the way forward: install that tool at the floor or newer.
The floor error is its own type, never a `*gitexec.GitError`, so no caller reads it as git's answer.
Its `doc.go` holds the audit result of D5.
- `internal/gitexec` runs `git --version` through its own exec core before its first git command, so `Run`, `RunStdin` and `RunGit` all return the floor error in a process whose git is below the floor.
  It logs every exec-core spawn, the probe included, through `internal/logger`; the import is legal, since `internal/logger`'s import closure no longer reaches `internal/gitexec`.
  The `internal/gitexec/gitexec.go` entry leaves `cmd/lyx/spawnobservability_test.go`'s allowlist, since its import-cycle reason no longer holds.
  gitexec exports the check, so a git spawn outside its exec core can call it.
- `internal/planglyph`'s `Delta` calls gitexec's exported check before `repo.DeltaGit`, whose git spawn runs inside quarry's `internal/gitsrc` and never reaches gitexec's exec core.
  Bound: `internal/gitkit`'s own git spawns stay unchecked; it is test plumbing no `lyx` command reaches.
- `internal/reedengine` exposes one check, run before the first tmux command of each resolved tmux path in the process, off Windows only, and logs its spawn through `internal/logger`.
  Every tmux spawn calls it: the `TmuxCmd` runner, the lifecycle server spawn and `reedcli`'s attach; the watchdog daemon reaches tmux through the same runner.
- Only a parsed result is cached for the process; a probe that cannot run returns its exec error and is retried on the next command.
  Bound: a long-lived daemon keeps its parsed result and is not rechecked after a git or tmux upgrade during its life.
- A version line that does not parse, such as a vendor or development build, is admitted with a Warn, in gitexec as in reed.
  Bound: only a parsed version below the floor is refused; nothing bypasses a parsed refusal.
- Tests reach the floor check through an injected version line, never a substituted git or tmux binary.
  A `TmuxCmd` built with an exec hook never reads or fills the process cache: it carries its own floor result, admitted unless the test sets a version line on it.
  So hook-driven tests run no probe and keep their exact call counts, such as `lock_test.go`'s zero-call and `overlay_test.go`'s one-call checks.
  A test that injects a version line into a process cache, gitexec's or reed's for the server spawn and attach, names that process-global state, resets the cache when it ends and runs without `t.Parallel`, per `PATTERN-test-speed`; Go starts a package's top-level parallel tests only after its serial ones finish, so no parallel test sees the injected line.

Rationale: every git command lyx runs goes through gitexec's exec core or the one quarry path above, and every tmux spawn through the reed sites above, so checking there covers every command that needs either tool, and a command that needs neither, such as `lyx board get` on a machine without tmux, pays nothing.
The floor equals the oldest versions lyx is proven on.

### D5 — Audit every git and tmux invocation

The Plan audits every git subcommand and flag lyx runs, in production code, test kits and `tools/`, and every tmux command, option and format feature reed and tmuxkit use, against git 2.43 and tmux 3.4.
Each feature newer than the floor is replaced by a form both the floor and newer versions run with the same behaviour, apart from D3's order of the non-prime sessions.
`internal/toolfloor`'s `doc.go` lists every feature lyx uses that arrived after git 2.20 or tmux 3.0, with the version each needs, so a later change checks a new feature against a recorded baseline.

Rationale: the board entry asks for the audit, and the suite catches only what it exercises: tmux formats were set but never checked as rendered.

### D6 — #744: the creel deliver tests synchronize on a probe that saw the post

Diagnosis: the failure reproduces on this host as a rare flake, so it is a race in the test, not a fault of the Ubuntu host.
`probeWaits(before + n)` counts any probe that went back to sleep.
A worker's wake channel holds one signal, so a wake left by an earlier post can start a probe that satisfies the count before the post under test is delivered.
The test then reads `typed` and sees only the owner's notice.

Fix: every subtest in `deliver_test.go` that waits by counting probe waits after a post, directly or through `tick`, waits instead for a probe whose snapshot was taken after the post, and then for the wait that follows it.
The marker is a pause read for the receiver's strand: `creelfake.Delivery` records each `Paused` call per strand, as it records `Idle` calls, and the test waits for one that starts after the post returned.
A live receiver's probe reads its pause after the strand read and before it snapshots the store, and no other reader asks about that strand's pause, so the marker excludes the sweep, `Store.post`'s strand read, the other store verbs and other receivers' workers.
The store's interrupt path is the one other pause reader, reached by `Store.Interrupt` and by an inject with its interrupt set, so a subtest that interrupts takes its marker after that call returns.
A probe of a receiver that is not live reads no pause and types nothing whatever it snapshots, so a check on a dormant receiver keeps its wait count.
This covers the subtests with more than one strand reader, such as `ProgramMailboxIsNeverReminded`, `HoldMutesOtherSendersLetsTheOwnerPassAndEndsAtExpiry` and `LogByIDListsTheRemindedEventsThatHoldTheID`.
A negative check waits for the marked probe and the wait after it, so a check that nothing was typed proves the probe under test ran; a post that leaves nothing typed stays pending, so that probe comes once the test advances the clock past the worker's wait.
A positive check waits instead for the outcome it asserts, the typed line or the journal event, since an unmarked stale-wake probe that snapshots after the post can deliver it and retire the worker, leaving no marked probe to come.
The scheduler is unchanged: a full wake channel already guarantees a probe that will see a post.

Rationale: per the operator, #744 is diagnosed and its cause fixed in this run, never deferred.

### D7 — The merge-lock tests prove the verb waits on the lock

Diagnosis, from the code: `launchBlockedOnLock` assumes the verb is parked on the weft write lock 150 ms after it starts.
Where git spawns are slow, as under the Cortex XDR scanning that `xdr-git-test-file-burst` reports on the same laptop, the verb's pre-lock guard stage still runs when the fixture rewrites a tracked file or switches branch in the warp checkout, and reads that as tracked dirt: `worktree dirty` instead of `ErrForeignMergeState`.
The Plan confirms it with a one-off diagnostic edit that never lands: slowing the verb's pre-lock path past the old wait must turn the current helper's result into `worktree dirty`; if it does not, git 2.43 stays the open cause.

Fix: `internal/fabricengine` gains an unexported test seam, set only from its `export_test.go`, that signals when a verb is about to wait for the weft write lock.
The signal is tied to one lock path, so parallel tests each wait only for their own verb.
Every verb `launchBlockedOnLock` launches, `MergeIn`, `Merge` and `MergeAbort` among them, reaches the seam at its lock acquisition.
`launchBlockedOnLock` waits for that signal, bounded by a real-time deadline, before it returns, then keeps its check that the verb has not finished.
Bound: the seam is nil in production and is called only where a launched verb acquires the weft write lock, in `merge.go` and `mergelifecycle.go`; it observes and changes nothing.

Rationale: a test that waits for a fixed time stands in for a state it does not check.

### Added after Discussion (arbiter, 2026-10-19): D6 refined: card 9's wait postcondition and inject coverage

Decision: Card 9's Intent is the mechanism for D6. awaitMarkedProbe ends when the probe after the marked one has begun (the pause read at mark+2), so the marked probe has finished and the wait after it has fired. That is how D6's 'waits for the marked probe and the wait after it' is met. Neither awaitMarkedProbe nor awaitTyped leaves the worker parked, so a later flip or clock jump re-marks through awaitMarkedProbe and never uses a bare tick or probeWaits(before + n). A single post made after e.start to a receiver with nothing pending since e.start keeps its probeWaits(1) wait, and so does a pre-start post. An inject made after e.start falls under D6 as a post does, with the per-subtest exceptions card 9 names. A positive check whose assertion reads the journal waits for the journal event, since awaitTyped returns before the probe journals.

Rationale: The plan review's round-3 fixer deferred this mismatch between D6 and card 9: card 9 states the postcondition and covers injects, and D6 covers neither. Checked against the code: the worker registers clock.After only after its probe returns (deliver.go run), Scheduler.wake creates a worker with an empty wake channel only for a receiver without one (deliver.go:72-95), Hold sends no wake (verbs.go:537-585), the inject path wakes the receiver (verbs.go:397), and a hold ends by the clock alone (pool.go:208).

### Added after Discussion (parent, 2026-10-19): tmux allow-set-title stays an optional above-floor feature

Decision: doc.go records allow-set-title (tmux 3.5) as a third above-floor form that lyx keeps as an optional feature: on a tmux 3.4 host reed Warns when setting it fails and the pane title stays one the agent can overwrite. Card 2 continues with this recorded; no plan revision.

Rationale: lyx already degrades here (internal/reedengine/spawn.go Warns on failure; naming_integration_test.go skips where the option is missing), so nothing on a 3.4 host breaks; the task goal is lyx working on Ubuntu 24.04, not reaching every tmux 3.5 nicety.

### Added after Discussion (operator, 2026-10-19): The tmux floor stays 3.4

Decision: TmuxFloor is never raised above 3.4 in this task; a feature found above the floor is kept optional with a fallback or replaced, never met by moving the floor.

Rationale: The task exists so lyx runs on Ubuntu 24.04 with tmux 3.4; raising the floor removes the point of the task (operator, 2026-10-19).

## Constraints

- No test installs git 2.43 or tmux 3.4: host conditions are reproduced through properties of lyx's own code, and the operator's host run after landing covers the real tools, per the operator.
- `PATTERN-lyx-temp-root`: run directories stay under lyxtemp's root, created 0700, owner-locked.
- `PATTERN-gitexec-checked-call` and `PATTERN-gitrepo-client-boundary`: the git version probe runs inside `gitexec`; `CherryPick`'s new calls go through `runChecked`.
- `PATTERN-spawn-observability`: both version probes, reed's tmux probe and gitexec's git probe, log their spawn, and gitexec logs every exec-core spawn, leaving the allowlist.
- `PATTERN-leaf-packages`: `internal/toolfloor` imports only the standard library and is listed in the background file.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: new tests take the lowest fitting tier, and reed's rendered-row check stays in its tagged integration file.
- `PATTERN-documentation-lifecycle`: the floor and the audit live in `internal/toolfloor`'s `doc.go`, and `docs/overview.md`'s module tree gains `internal/toolfloor`.
- PATTERN.md stands one byte under its headroom line, so this task adds no PATTERN entry, although CLAUDE.md asks one for a new cross-cutting invariant: the mechanical headroom check wins, and `## Open risks` names where the floor rule lives instead.
- Behaviour on newer git and tmux stays the same: the same pick result, an empty-from-start commit refused as before, the same session row apart from the prime's place.
  On tmux 3.4 the other sessions follow in name order, where newer tmux keeps session-id order, since the floor's `S` loop has no sort.

## Auto-mode assumptions

- The floor is git 2.43.0 and tmux 3.4, the NORCE host's versions, not a lower version the audit might allow.
- The floor check runs at each tool's first use, not in the root command.
- An unparseable version line is admitted.

## Open risks

- git 2.43 and tmux 3.4 behaviour is proven on the NORCE host only after landing, per the operator; a failure there becomes a follow-up.
- D7's cause rests on the code and the slowed reproduction, since the plain stress run stays green here; if the host still fails after the fix, the cause lies elsewhere.
- `#{client_session}` inside an `#{S:}` loop is checked rendered on this host's tmux only; tmux 3.4's man page lists it as a client format available in status formats.
- A 10-digit uid with a `TMPDIR` longer than `/tmp` can still pass the socket limit; tmuxkit's existing error names `TMPDIR` as the remedy.
- The rule that a git or tmux feature must exist at the floor lives in `internal/toolfloor`'s `doc.go` and in review, not in PATTERN.md, until the overview has room.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d3-boot-cannot-mark-existing-prime (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-1-review.md; fixer report: _lyx/reviews/plan/round-1-fixer-report.md.
  - card7-attach-preflight-docs-stale (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- A run directory's name is `r` plus six hex digits, and tmuxkit's socket-path check admits a run directory of that name under uid 4294967295.
- Run directory creation retries a taken name and fails, naming the root, once the attempt bound is spent.
- `CherryPick` of a commit whose change HEAD already carries returns nil with HEAD and index unmoved, no cherry-pick in progress, and uncommitted working-tree changes kept.
- A conflicting pick returns the error with the pick in progress, and `CherryPickAbort` restores HEAD.
- `CherryPick` of a commit that was empty from the start returns an error with the pick in progress, and `dropSeedCommits` then returns `errDropReplayFailed` with the start tip restored.
- No `cherry-pick` argv lyx runs carries `--empty`, and the Bolt seed-commit drop tests pass.
- Reed's rendered session row shows each reed session exactly once, the prime's session first, then each other reed session once, in the `S` loop's order, the client's session lit with its full name, and no comma between buttons.
- A prime session created by an older binary is marked after an attach to it, and a prime session a fresh boot spawns is marked by that boot, and a standalone session is never marked.
- Every `#{S:}` loop in reed's formats has a single format.
- The floor parser admits git 2.43.0, a newer git, a Windows git (`2.45.1.windows.1`), an Apple git at or above the floor (`git version 2.50.1 (Apple Git-155)`), and tmux 3.4, `3.4a` and newer.
  It refuses git 2.42, a below-floor Apple git (`git version 2.39.5 (Apple Git-154)`) and tmux 3.3a with an error naming the tool, the found version, the floor and the way forward, and admits a line it cannot parse.
- With a below-floor version line injected, `Run`, `RunGit` and gitexec's exported check all return the floor error, which is no `*GitError`; the probe runs once per process after a parsed result and again after a probe that could not run, and an unparseable line is admitted with a Warn.
- With a below-floor tmux version line injected, the `TmuxCmd` runner, the lifecycle server spawn and reedengine's exported check each return the floor error before running tmux.
  `planglyph.Delta` and `reedcli`'s attach, whose packages cannot reach the process caches from their tests, call the exported check before they spawn.
  The probe runs once per resolved tmux path, and an unparseable line is admitted with a Warn.
  A Windows-only test file pins that the check runs no probe on Windows.
- `internal/toolfloor`'s leaf enforcement test pins its import set to the standard library.
- `internal/toolfloor`'s `doc.go` lists every git and tmux feature after git 2.20 or tmux 3.0 that the audit found, each with its version, and none is newer than the floor.
- The deliver subtests pass under `-count=1000 -cpu=1,2,4`, a check that nothing was typed waits for a probe that saw the post, and a check that something was typed waits for that outcome.
- The merge-lock tests wait for their verb's lock signal, never a fixed time, and parallel tests each wait for their own verb.
- `lyx gate test ./...` and `lyx gate test --tags integration ./...` pass on this host.

## Notes for the plan writer

- The #739 diff in the board entry's #745 section, host-tested there, is the starting point for D1, including its tests and the PATTERN background edit.
- For D6, `tick` and the direct `probeWaits(before + n)` callers carry the counting shape; `sweepStep` counts sweep waits and is not part of the race.
