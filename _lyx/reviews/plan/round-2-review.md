---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/07-creel-delivery-teardown.md:16
    summary: Card 7 plans TestWorktreeTornDown, a per-symbol test of an unexported helper, on the false premise that a gone gitdir never reaches Strands
    fix: >-
      In card 7 drop the Create line `plan:internal/creelcli#TestWorktreeTornDown` and the Tests bullet that begins "a gone directory or gitdir never reaches the resolve through `fabricengine.List`".
      Add to the rows of `TestHubDelivery_SkipsAWorktreeThatNoLongerResolves` one for the pair's `.git` replaced by a gitfile whose `gitdir:` line names a missing directory: git still lists the worktree, so `Strands` skips the pair and logs the skip at Info.
      Add one bullet saying `worktreeTornDown` gets no test of its own, since the status and wire rows reach its gone-directory mark, the missing-gitdir row its gone-gitdir mark, and the two Warn rows (empty `.git` directory, gitfile naming an existing empty directory) its two false answers.
      Checked in memory with `lyx loom validate-plan --overlay`: the revised card 7 reports no finding beyond the baseline.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/plan/05-reed-pane-elsewhere.md:34
    summary: Card 5 logs a failed server-wide pane lookup at Warn on every Status and reconcile, a polled path, with no episode guard
    fix: >-
      In card 5's `Engine.strandsElsewhereLocked` paragraph, replace "A failed lookup is logged at Warn and returns nil" with: a failed lookup goes through `e.calleeFailures.Fail("pane placements", "reed: could not look up the panes of the whole server, clearing the bindings the session lacks", "socket", e.Socket(), "session", e.SessionName(), "err", err)` and returns nil, and a successful lookup calls `e.calleeFailures.Recover("pane placements", "reed: the server-wide pane lookup works again", "socket", e.Socket(), "session", e.SessionName())`.
      That logs Warn at the first failure, Debug while it repeats and Info at recovery, as `adoptPaneGenerationLocked` does for its tmux read.
      In the `TestStatus_ReportsOwnWindowStrandsFromTheSessionListing` failing-lookup row, add that a second failing `Status` logs no second Warn.
      No target changes, so no overlay is needed.
  - id: F3
    severity: BLOCKING
    class: design
    location: _lyx/plan/07-creel-delivery-teardown.md:36
    summary: Card 7 logs every skip of a worktree at Warn or Info on each Strands call, and a persistent broken link repeats the Warn on every creel probe
    fix: >-
      In card 7, add `internal/creelcli#hubDelivery` to the Edit list and, after the sentence that fixes the Info and Warn levels, add: creel's delivery workers and sweep call `Strands` on every probe, so `hubDelivery` remembers, under a mutex since the workers call it concurrently, the worktree paths it has logged a skip for; the first skip of a path logs at the level above, a later skip of the same path logs at Debug, and a path that resolves again is forgotten.
      In `TestHubDelivery_SkipsAWorktreeThatNoLongerResolves`, extend the empty-`.git`-directory row so a second `Strands` call logs the skip at Debug and not at Warn.
      Checked in memory with `lyx loom validate-plan --overlay` together with F1: the combined card 7 reports no finding beyond the baseline.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/08-orch-request-marker-lock.md:69
    summary: Card 8's ImpactSummary says ClearCycleRequest has three production callers, but it has four
    fix: >-
      In card 8's ImpactSummary replace "`ClearCycleRequest`'s three production callers are all on this card" with "every production caller of `ClearCycleRequest` is on this card".
      The callers are the stale removal and the cycle start in `Watcher.tickIdle`, `Watcher.removeSatisfiedRequest`, and `orchCLI.requestCycle`.
---

### [BLOCKING] Card 7 plans a per-symbol test on a false premise

**Location:** `_lyx/plan/07-creel-delivery-teardown.md:16` and the Tests bullet at line 53.

**Issue:** `PATTERN-test-economy` tests an unexported helper through the surface that uses it, and a new test must cover a behavior no existing test covers.
Card 7 creates `TestWorktreeTornDown` for `worktreeTornDown`, justified by "a gone directory or gitdir never reaches the resolve through `fabricengine.List`, which marks such a worktree prunable".
That holds for a gone directory only, and the card's own status and wire rows already reach that mark through `os.Stat`.
It fails for a gone gitdir: `fabricengine.List` parses `git worktree list --porcelain`, whose `prunable` mark follows the admin directory's `gitdir` file, which names the worktree's own `.git`.
I checked it with git 2.53 on a scratch repository: a worktree whose `.git` gitfile names a missing directory is listed with no `prunable` line, `lyxcwd.ResolveWorktree` then returns `ErrNotAGitRepo`, and `git worktree repair <path>` from the prime rewrites the link.
So the missing-gitdir mark is reachable through `Strands`, as a sibling of the card's existing gitfile row, and every row of the new table is covered by a `Strands` row.
The test is a per-symbol test whose only stated reason is wrong.

### [BLOCKING] Card 5 logs a repeating failure at Warn in a polled path

**Location:** `_lyx/plan/05-reed-pane-elsewhere.md:34`.

**Issue:** `PATTERN-log-levels` binds logging in a loop or poll to `internal/logger`'s Level policy: a failure that repeats each tick logs Warn once per episode, Debug while it persists, and Info at recovery.
`strandsElsewhereLocked` runs inside `Engine.Status` and every public op's reconcile, which the orch watcher, shuttle's wait and creel's delivery poll, and it runs on every such call while a bound pane is absent from the session.
The card logs its failure at Warn on each call, so a lookup that fails persistently floods the log and the trace file.
That is plausible where a multiplexer lacks `list-panes -a`: reed supports psmux, and `internal/reedengine/probe.go` records that psmux lacks some verbs.
Reed already guards the same shape: `adoptPaneGenerationLocked` reports its swallowed tmux read through `e.calleeFailures`, a `logger.EpisodeSet`.

### [BLOCKING] Card 7's skip line repeats on every Strands call

**Location:** `_lyx/plan/07-creel-delivery-teardown.md:36`.

**Issue:** `PATTERN-log-levels` forbids a repeating failure from logging at Warn on each tick.
`Strands` is called from creel's delivery workers on every probe, from its sweep and from its verbs, so a worktree with a persistent cause that is no teardown, a broken gitfile link or an empty `.git` directory, logs the Warn line on every call until an operator repairs it.
Card 7 fixes the levels, Info for a teardown mark and Warn otherwise, and says nothing of repeats.
The decision record's levels stay as they are: the first skip of a path logs at them, and only repeats drop to Debug, as the Level policy requires.
Creel's sweep already dedupes its own listing failure by error text for the same reason.

### [LOW] Card 8's ImpactSummary miscounts the callers

**Location:** `_lyx/plan/08-orch-request-marker-lock.md:69`.

**Issue:** The ImpactSummary states a count that is wrong: `ClearCycleRequest` is called at `internal/orchengine/watcher.go` in `tickIdle` twice (the stale removal and the cycle start), in `removeSatisfiedRequest`, and at `internal/orchcli/refresh.go` in `requestCycle`.
The conclusion, that every caller is on this card, is right; a count in prose goes stale and here is already off.

## Rejected

- Focus, card 7: the way forward the Warn line names ends the skip in the rows that name it.
  For a gitfile whose `gitdir:` names an existing empty directory, which is the planned row, I ran `git worktree repair <path>` from the prime of a scratch repository: it printed an error for the path pass, exited 1 and still rewrote the gitfile to the admin directory, because the prime's `worktrees/<id>/gitdir` records the worktree.
  A gitfile naming a missing directory is repaired the same way.
  For a worktree whose `.git` is a directory, the planned Warn line names the cause and no step, and `git worktree repair` refuses it with `.git is not a file`, so naming no step is right.
  The card does not promise a repair where the admin directory is gone.
  Not a defect.
- Focus, card 11: a failed clear of `ResumeNoticed` in `withdrawQueuedCompact` is retried.
  The failing tick saves `Pending` nil and `ResumeLineWithdrawn` set before the boundary is consumed, and `Watcher.Run` keeps ticking on errors up to its consecutive-failure bound; the next tick finds the same boundary through `CompactedSince`, re-runs `ClearResumeNoticed` with the resume-line entry, and completes the phase on success.
  A restarted watcher reads the same saved state.
  The planned test row makes the record unwritable, fails the tick, restores it and asserts the following tick clears the mark and moves to resuming, so the retry is reached.
  Not a defect.
- Focus, card 10: no planned assertion pins a stencil's static prose.
  The test drops every literal of both note requests and keeps the Go-filled note template path and `PendingHandoff` path plus the hard-versus-soft `defer` rule, which `PATTERN-test-economy` allows as a rule across a stencil family.
  The stencil rewordings are guarded by review, as the card says.
  Not a defect.
- Card 6 keeps the name of a dead holder that has child strands, which the decision record's D6 does not list among the exceptions.
  Reed's ordinary removal refuses a non-leaf without recursion, so the choice follows from D6's "ordinary strand removal" and a test row pins it; not raised.
- Card 1 plans `TestParseWindowPanes` for an unexported parser.
  The sibling parser has `TestParsePaneList` as precedent in the same package and no existing test calls this one; not raised.
- Card 2 reads the clock tick rate through cgo and the stat start time again, though `internal/proc` has `StartTime` and a fixed `userHZ`.
  D1 names the cgo read, so the plan follows the decision; not raised.
- The tmux premise of card 1 holds: with tmux 3.6 on this host, `list-panes -F` passes the unit separator through verbatim under `C.UTF-8`, `C` and `POSIX` locales, and `#{pane_start_command}` is empty for a pane of a session with no `default-command` pin.
