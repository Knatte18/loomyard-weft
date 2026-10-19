---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md:70 (D4 gitexec bullet) and :126 (Constraints, PATTERN-spawn-observability)
    summary: D4 leaves gitexec's new git --version probe unlogged under an exemption whose structural bar no longer exists, so the probe spawn breaks PATTERN-spawn-observability
    fix: In D4's gitexec bullet, replace the sentence that leaves the probe spawn unlogged with one that has gitexec log the probe spawn through internal/logger, and change the Constraints bullet for PATTERN-spawn-observability to say both probes log their spawn. In D4's unparseable-version bullet, replace "silently in gitexec, which cannot log" with a Warn in gitexec as in reed. The import is legal because internal/logger's import closure (dotgit, lyxcwd, lyxdirs, proc) no longer reaches gitexec. If the designer finds a real structural bar instead, the Decision must name that bar in place of the stale one.
  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md:39-40 (D2) and :131 (Constraints, same pick result)
    summary: D2's stopped-empty test also matches a commit that was empty from the start, so --skip now drops a non-seed commit that newer git refuses today, against D2's as-before claim and the same-pick-result Constraint
    fix: >-
      Make one choice and write it into D2 and the Constraints bullet. Either (a) accept the drop: D2's Bound names an empty-from-start commit as dropped too, the Constraint names this as the one change on newer git, and an acceptance criterion pins it. Or (b) keep today's result: before --skip, CherryPick refuses when the picked commit's tree equals its first parent's tree, through one more runChecked CLI read, and returns the error with the pick in progress for CherryPickAbort.
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md:54 (D3) and :154 (acceptance criterion)
    summary: tmux 3.4's S loop walks sessions in name order with no sort option, so D3's promise of the other sessions in session-id order, and the matching acceptance criterion, cannot hold on the floor
    fix: In D3, replace "every other session in session-id order" with "every other session in the S loop's own order, which is session name on tmux 3.4 and session id on newer tmux". In the acceptance criterion at line 154, replace "then the others in session-id order" with "then each other reed session once, in the S loop's order". Add the order difference to the Constraints bullet on newer-tmux behaviour, since D3 now gives a different order on the floor than on newer tmux.
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md:100-101 (D6 Fix)
    summary: the fake delivery's Strands read takes no caller, and the sweep, Store.post and the other store verbs, other receivers' workers and the test's own reads all call it after a post, so D6's marker does not single out the probe under test and the stale-wake race survives
    fix: Rewrite D6's marker sentence so the marker identifies the receiver's own probe. For example, the fake delivery records each Idle or typing call per strand, which only the receiver's worker makes, or the test waits for a Strands read with no other reader running, naming how it excludes the sweep, posts and verbs and other workers. Then name the subtests with more than one reader, such as ProgramMailboxIsNeverReminded, HoldMutes and LogByID, as covered.
  - id: F5
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md:81 (D4 Rationale)
    summary: D4 claims every git command runs through gitexec's exec core, but production planglyph.Delta spawns git through quarry's internal/gitsrc, bypassing the floor check, and D4 states no bound for it
    fix: In D4's Rationale, replace "every git command runs through gitexec's exec core" with the true claim, and add a bullet for the quarry path. Either planglyph.Delta calls the gitexec floor check, through an exported check function, before repo.DeltaGit, or D4 states the bound that quarry's git spawns go unchecked and that a process reaching them has, or has not, already run a gitexec command.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md:73-79 (D4 reed bullet, caching bullet, injection bullet)
    summary: D4's floor result is cached per process and the version line is injected for tests, but the exec hooks are per test, so the tmux -V probe lands in whichever hook-driven test runs first, and an injected below-floor line fails every parallel test in the binary
    fix: Add to D4 how the cache and injection are scoped in tests. Either the floor check and its cache hang off the Engine or TmuxCmd, and the gitexec equivalent off an injected checker, so a test seeds its own result; or hook-driven tests start from a seeded parsed result and the one test that injects a below-floor line names the process-global state and runs without t.Parallel, per PATTERN-test-speed. Exact call-count tests then keep their counts, such as reedengine's lock_test.go zero-call check and overlay_test.go one-call check.
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md:72 (D4 reed bullet) against :149-165 (Acceptance criteria)
    summary: no acceptance criterion covers D4's tmux side; only the parser and gitexec's Run and RunGit are pinned
    fix: Add an acceptance criterion that, with a below-floor tmux version line injected, the TmuxCmd runner, the lifecycle server spawn and reedcli's attach each return the floor error before running tmux. It should also check that the probe runs once per resolved tmux path, that no probe runs on Windows, and that an unparseable line is admitted with a Warn.
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md:111 and :117 (D7) against :163 (acceptance criterion)
    summary: the acceptance criterion needs the verb's pre-lock guard stage slowed, which no Decision introduces, and D7 bounds its only seam to an observing signal at the lock acquisition, which cannot slow the guards that run before it
    fix: Add to D7 the mechanism that slows the pre-lock path, such as a second unexported export_test.go seam called at the start of the guard stage. Widen D7's Bound to name both seams, their call sites and what each may change, and replace "called at that one point only" to match, since the launched verbs take the lock at more than one site (merge.go and mergelifecycle.go). Or drop the slowed-path clause from the acceptance criterion and keep the slowed reproduction as the Plan's one-off diagnostic step in D7.
  - id: F9
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md:53 (D3 marking)
    summary: the attach pre-flight re-pins status-format[1] through pinGeometryOptionsLocked without booting, so a prime session from an older binary that is only attached draws in the second loop, not leftmost, until a boot
    fix: In D3, set the mark wherever pinGeometryOptionsLocked runs, at boot and in the attach pre-flight, for a prime geometry, and replace "whenever it creates or boots onto the prime's session" with that wording. Add attach to the acceptance criterion on marking an older binary's prime session.
  - id: F10
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md:61 (D3 Rationale) and :98 (D6 Diagnosis)
    summary: D3's rationale restates a rejected alternative, which the spec keeps out of the decision record, and D6 carries a host-history sentence the plan writer does not act on
    fix: Delete line 61 (Marking the prime by an option rather than by its name...); the support log's Rejected alternatives already holds it. Delete line 98 (A slower or busier host widens the window...), or move it to the support log's Turn 3.
  - id: F11
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md:157 (acceptance criterion)
    summary: the criterion says the floor parser admits an Apple git, but stock Apple git is often below 2.43 and would be refused by D4
    fix: Replace "an Apple git" with an Apple git line at or above the floor, for example git version 2.50.1 (Apple Git-155), and add a below-floor Apple line to the refused cases.
---

### [BLOCKING] gitexec's version probe rests on a stale logging exemption

**Location:** `_lyx/discussion/decision-record.md:70`, `:76-77`, `:126`

**Issue:** D4 says the gitexec probe spawn "is unlogged under gitexec's existing reason for not importing the logger", and that an unparseable git line is admitted "silently in gitexec, which cannot log".
The recorded reason in `cmd/lyx/spawnobservability_test.go:72-75` is "internal/logger imports internal/lyxcwd, which imports internal/gitexec, so importing logger here would close an import cycle".
That cycle no longer exists.
`go list -f '{{.Imports}}' ./internal/lyxcwd` lists only `dotgit` among internal packages, and `go list -deps ./internal/logger` reaches only `dotgit`, `lyxcwd`, `lyxdirs` and `proc`.
`pattern/PATTERN-spawn-observability.md` exempts only "a site structurally barred from importing `internal/logger`", so the new spawn D4 adds is not exempt.
The Decision leans on the allowlist entry's text without checking whether its premise still holds, and the Constraints bullet repeats it.

### [MEDIUM] D2 drops an empty-from-start commit that newer git refuses today

**Location:** `_lyx/discussion/decision-record.md:39-40`, `:131`

**Issue:** The stopped-empty test (a pick in progress, nothing unmerged, index equals HEAD) holds on git 2.43 for both kinds of empty pick.
The pick state is written before the commit step fails, and the `--skip` advice dates from 2.23.
It cannot tell them apart, though.
On git 2.53, `git cherry-pick --empty=drop <commit that was empty from the start>` exits 1 with `CHERRY_PICK_HEAD` set, because `--empty=drop` only drops a commit that became empty.
Today `dropSeedCommits` (`internal/fabricengine/destroy.go:1769-1779`) then aborts and returns `errDropReplayFailed`, which Bolt reports as `diverged` (`bolt.go:295`).
Under D2 the same state passes the test, `--skip` runs and the call returns nil.
That contradicts "as before and on every git version" and the Constraint "the same pick result".
The `--skip` bound itself holds: such a commit carries no change, and `--skip`'s reset-merge rollback keeps the working-tree changes `reset --keep` carried.

### [MEDIUM] tmux 3.4 cannot draw the other sessions in session-id order

**Location:** `_lyx/discussion/decision-record.md:54`, `:154`

**Issue:** D3's premise holds: tmux 3.4's `format_loop_sessions` takes one format, and `S` parses only as a no-argument modifier.
`#{client_session}` also resolves inside the loop, since each item's tree is built with the client.
But the 3.4 loop is `RB_FOREACH` over a tree whose `session_cmp` is `strcmp` on names, so it walks sessions in name order.
tmux 3.6 sorts the loop by session id by default.
The board entry's #748 says so too: "`S:` also walks sessions in name order with no sort option in 3.4".
D3 adds no sort, so "every other session in session-id order" and the acceptance criterion fail on the floor host.
The operator's #740 expectation asks only for the prime leftmost with the pair sessions to its right.

### [MEDIUM] D6's marker does not identify the probe under test

**Location:** `_lyx/discussion/decision-record.md:100-101`

**Issue:** The diagnosis holds.
`probeWaits(before + n)` counts any probe wait, a worker's one-slot wake channel can start a probe from an earlier post, and every loop iteration records a wait through `clock.After`.
The fix keys on "a strand read by the test's fake delivery that starts after the post returned".
But `creelfake.Delivery.Strands` takes no caller, and other readers call it after a post:
- the sweep's `sweepOnce` (`sweep.go:114`) once its timer fires;
- `Store.post` itself through `readStrands` (`verbs.go:63`), for a second post such as HoldMutes' 4097-byte one;
- the other store verbs a test calls, such as `readBodies` (`verbs.go:271`, `:438`);
- other receivers' workers, as in ProgramMailboxIsNeverReminded (`deliver_test.go:457-459`).

Any of these can set the marker while a stale-wake probe supplies the next wait, so the race the Decision fixes can survive in the subtests it targets.

### [MEDIUM] A production git spawn bypasses gitexec's floor check

**Location:** `_lyx/discussion/decision-record.md:81`

**Issue:** `internal/planglyph/delta.go` calls quarry's `repo.DeltaGit`, and its own comment says quarry's `internal/gitsrc` "runs exec.Command("git", ...)".
That spawn never reaches `gitexec.runCore`, so the Rationale's "every git command runs through gitexec's exec core" is false.
A process whose first git use is that path gets a below-floor failure from quarry, not the floor error.
The focus directive asked whether any git spawn path is left outside the check; this one is, with no stated bound.

### [MEDIUM] Process-wide cache against per-test exec hooks and injection

**Location:** `_lyx/discussion/decision-record.md:73-79`

**Issue:** The reed probe goes "through the runner's exec hook" in tests, but its result is cached per process and per tmux path, while each test installs its own hook.
In a test binary, the `-V` probe lands in whichever hook-driven test first runs tmux for that path, and that depends on run order.
Tests that pin exact call lists then become order-dependent, such as reedengine's `lock_test.go` zero-call check and `overlay_test.go` one-call check.
A scripted fake that answers an unscripted `-V` with nothing turns the probe into an unparseable line, so those tests also run the admit-with-Warn path.
In the other direction, an injected below-floor version line is process-global, so it fails every concurrent `t.Parallel` test in the same binary that runs git or tmux.
D4 does not say how either is scoped.

### [MEDIUM] No acceptance criterion for the tmux floor check

**Location:** `_lyx/discussion/decision-record.md:72` against `:149-165`

**Issue:** D4's reed bullet has its own behaviour:
- the check runs before the first tmux command of each resolved path, off Windows only;
- it covers the `TmuxCmd` runner, the lifecycle server spawn (`lifecycle.go:458`) and the attach (`reedcli/attach.go:120`, `attach_unix.go`);
- an unparseable line gets a Warn.

The acceptance list pins only the parser and gitexec's `Run` and `RunGit`, so none of the tmux behaviour has a criterion.

### [MEDIUM] D7's acceptance criterion needs a seam D7 forbids

**Location:** `_lyx/discussion/decision-record.md:111`, `:117`, `:163`

**Issue:** The diagnosis follows from the code.
`MergeIn` runs `pairDirtyReason` and the other guards before `AcquireWriteLock` (`merge.go:150-198`), and `setupConflictingDivergence` commits on the warp checkout.
The fix also addresses the cause, since the helper now waits for the lock signal rather than a fixed time.
But the acceptance criterion requires the tests to pass "with the verb's pre-lock path slowed past the old fixed wait".
Slowing the guard stage needs a hook before the guards, and D7 declares only a signal at the lock, bounded as "called at that one point only; it observes and changes nothing".
The launched verbs also take the weft write lock at several sites (`merge.go:195`, `:399`, `mergelifecycle.go:286`, `:382`), not one point.

### [LOW] Attach re-pins the new format without marking an older prime

**Location:** `_lyx/discussion/decision-record.md:53`

**Issue:** `pinGeometryOptionsLocked` sets `status-format[1]` both at boot and in the attach pre-flight, as its own doc comment says (`windowsize.go:56`).
D3 sets the prime mark only at create or boot.
So an older binary's prime session that is attached, not booted, gets the two-loop format with no mark, and its button falls into the second loop until the next boot.

### [NIT] Rejected-alternative and history prose in the decision record

**Location:** `_lyx/discussion/decision-record.md:61`, `:98`

**Issue:** Line 61 argues against the rejected name-comparison alternative, which `loom-discussion-spec.md` keeps out of `## Decisions`; the support log already records it.
Line 98 explains the NORCE host's repeat failure, which the plan writer does not act on.
The diagnosis sentences around it carry the requirement and stay.

### [NIT] Apple git in the admitted set

**Location:** `_lyx/discussion/decision-record.md:157`

**Issue:** The stock Apple git (`git version 2.39.5 (Apple Git-154)`) is below the 2.43.0 floor, so D4 refuses it.
"admits ... an Apple git" reads as admitting that, when the case is the vendor suffix on a version at or above the floor.

## Rejected

- existing-paths: an older lyx binary still running on the shared server re-pins the old two-format `status-format[1]`. Rejected: production runs one installed binary, and every global format change shares this window until the old process exits. It is not specific to D3, and the record need not bound it.
- bound-breaker: D7's "one point" wording, filed as its own LOW finding. Rejected as a separate finding: it is merged into F8, whose fix rewrites that bound.
- interactions: D3 against D5's "same behaviour on floor and newer" rule. Rejected as a separate finding: it is the same defect as F3, and F3's fix states the order difference.
- D1's attempt bound has no number. Not flagged: D1 states that a bound exists and what happens when it is spent, and the notes point at the #745 diff, which fixes it at 10000.
- The PATTERN.md headroom departure in Constraints. Not flagged: `PATTERN.md` is 15563 bytes against `HeadroomBytes` 15564, so the claim is true, and the record states the conflict with CLAUDE.md openly as the rules require.
