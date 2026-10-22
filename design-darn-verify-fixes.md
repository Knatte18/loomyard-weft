# Darn-Write stops looping on verify failures it cannot own, and the flaky reed boot test behind them

`darn-verify-fixes` · Task · bug, enhancement, batten, gate, loom, shed, reed · running · Merge-Bouncer

# Goal

A verify failure Darn-Write cannot own, one in code the task did not touch or one the parent has already fixed, no longer loops through Darn-Write and escalates;
the flaky reed test behind it is made deterministic, and the impacted set stops picking a package the diff cannot reach.

# Re-scoped 2026-10-22 (operator decision)

Landed since this task was written, so out of scope:

- `de6b8465f` retired landing's `publish_verify` and darn's `verify`: every verify site, Publish included, now runs its gate's tier list through `lyx gate test --tier N --wait`, and Publish runs the `publish` gate (the loomyard hub's `tier2`, tmux).
- `a87a8a9aa` fixed the `TestOpenWindow_RealTmux/ClosesWhenItsCommandExitsAtOnce` flake (#765's failing test).
- `01ccaf6de` scoped Darn-Write's and Darn-Burler-Fix's gates to the diff's impacted set; Merge-In and Finalize still run the whole tree.
- `7ac8070a8` made the reed watchdog's stop marker work without inotify (#785's test half).

# Still open

## 1. `TwoWorktreesBootedConcurrently` is flaky (#714)

`internal/reedengine` `TestBoot_TmuxConf/AppendingFileIsSourcedOncePerServerStart/TwoWorktreesBootedConcurrently` failed a Publish verify of a task that did not touch `internal/reedengine`, with `run list-commands: exit status 1: server exited unexpectedly`: a race in booting two tmux servers at once.
Make it deterministic, or retry or isolate the tmux server start it races on.

## 2. The writer and a tier it may not run (#714)

In #714 the Darn-Write session could not reproduce the failure, since tmux-tier go runs were refused in a darn session, so it handed off with no commit and the run escalated.
Check whether a darn session can now run the failing tier through `lyx gate test --tier N`; if it can, nothing is left here; if not, give it that way or have it escalate at once instead of looping through the Darn cycle.

## 3. A failure the parent already fixed

Seen 2026-10-20 (`arbiter-verdict-quoting`): Publish failed on a test outside the branch and batten sent the run back to Darn-Write.
The fix landed on main meanwhile, but Darn-Write's verify kept failing on the task branch, which takes main only at Merge-In; the orch had the writer run `lyx fabric merge-in main` by hand.
When a verify failure is sent back to Darn-Write and the parent has moved since, merge the parent in before the verify re-runs.

## 4. The Publish halt and code the task did not touch (#765)

A Publish failure in code the task did not change sends the run back to Darn-Write, which spends gate attempts on a failure it cannot own.
The halt's way forward should tell a failure in code the task touched from one in code it did not.

## 5. The gate findings file outlives the halt (#765)

After `lyx loom goto --to Darn-Write` and `lyx loom resume`, Darn-Write halted with "gate did not pass after 3 attempts" and the `gate-findings.md` its halt note named was gone.
Keep the gate findings file until the halt is cleared.

## 6. The impacted set picks an unreachable package (#785)

A gate for a change confined to `internal/planglyph` ran `internal/reedcli`'s tests, although no changed file lies in that package's test closure, so an unrelated test blocked the task.
Find why `internal/impactset` selected `internal/reedcli` for that diff and fix the selection.

Provenance: tasks `stencilstore-dev-warning-noise`, `arbiter-verdict-quoting`, `create-target-transient`; issues #714, #765, #785.
