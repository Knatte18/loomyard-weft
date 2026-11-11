```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, here is what depends on it.

**Per change: the set found, covered or not**

1. **Waiter now spawns `--loop-keeper=<id>` instead of `--loop-detached=<id>`.**
   - The set: `runUntilStop`, `spawnLoop`, `LoopSpec.Spawn`, the `loopIDOf` test fake, the helper roles in `loop_integration_test.go`, the arming-refusal tests in `loop_arming_integration_test.go`, `loop_test.go:124`, `reportArmFailure` in `loomcli/cli.go` and `shedcli/cli.go`, the flag registration in `step.go`.
   - A repo-wide search for `loop-detached` or `loop-keeper` found no other reader. The only match outside Go files is the `shedverbs/doc.go` text, which the diff updated.
   - Covered, except finding 1 below.
2. **The waiter's exit mark is now `keeper process <pid> exited`; the keeper writes `process <pid> exited`.**
   - The set: `loopExitEvidence`, which joins marks by prefix and so reads both, and the `loopwait_test.go` and unix integration-test assertions.
   - No stencil, spec or `.md` file quotes the mark text, and no search hit for `loop exit: not recorded` or `process <pid> exited` outside Go. Covered.
3. **A new keeper process registers through `OnStart` and ends through `runproc`.**
   - The set: `loomcli/looparm.go:60` (`registerRunProcess`, which uses the calling process's own record), `battencli/arm.go:490` (`runproc.SelfRecord`), the `runproc` end path (SIGTERM, then SIGKILL after `endGrace`), and `PATTERN-child-processes`.
   - The registration reaches the keeper, because `OnStart` runs in it and both arming sites register the current process. The keeper catches SIGTERM, so `runproc` has to wait out `endGrace` and then SIGKILL it. The summary's claim that the keeper ends "through the loop's end" holds only when the loop itself is also registered and killed. It is, because `runLoop` runs `OnStart` too. Covered, except finding 2 below.
4. **`DetachedLoopID` replaces direct `LoopDetachedFlag` reads.** The only two flag readers are `reportArmFailure` in `loomcli` and in `shedcli`; both are updated. `step.go` itself uses it. Covered.
5. **`deadLoopStop` and `loopExitStop` now key on `hasChild()` rather than `inFlightAt`.**
   - The set: `StatusMoved`, `stderr_path`, `trace_copy`, the interrupt hook's trace file, the `deadLoopStop`, `loopExitStop` and `inFlightAt` doc comments, the `doc.go` text, and the dead-loop table in `loopwait_test.go`.
   - All are updated consistently. The `loop.status_moved` text in `doc.go` is unchanged and still matches. Covered.
6. **Docs and PATTERN.**
   - `PATTERN-shed-driver.md` now lists `loopkeeper.go`.
   - `shedrun.LoopLog`, `battenLoopSpec`, `loomcli/doc.go` and `shedverbs/doc.go` are updated.
   - `LoopSpec.Spawn` and `doc.go` are not updated; see findings 1 and 2.
   - The `shed-template-driver.md` stencil says only "a detached loop" and is still accurate.
   - No Pattern-Departure trailers appear in the log, and none are needed. The directive's declared-changes list names no PATTERN entry. Covered.

**Findings**

1. **LOW, consistency.** `LoopSpec.Spawn`'s doc is stale (`internal/shedverbs/loop.go:100`).
   - It reads "Spawn starts the detached loop with the given command line ... returns a channel closed when it exits".
   - After the diff it starts the keeper, and the channel closes when the keeper exits.
   - `spawnLoop`'s own doc was updated, so this doc is the one that was missed.

2. **LOW, consistency.** The `OnStart` contract contradicts the new keeper behaviour in two places.
   - `internal/shedverbs/loop.go:88-90` says "A loop that refuses before its first step never runs it" and then adds "It also runs once in the loop's keeper". The keeper runs it after the loop process starts, even if that loop then refuses at the lock or the start check.
   - `internal/shedverbs/doc.go:228` still says "A loop that refuses before its first step, and a dead loop the waiter reports, never run OnStart or OnExit". It does not say that the keeper has already run `OnStart` by then.
   - Both statements should say that the keeper's `OnStart` run is independent of whether the loop refuses.

I found no unrecorded caller or assertion of the changed signatures, outputs or flags beyond these.

**Tool calls (2)**
1. `grep -rniE` over `.md`/`.tmpl`/`.txt`/`.yaml` and over Go non-tests for loop-spawn phrases, to find docs, stencils or other code that describe the loop spawn or the exit marks. My context had only the Go files in the manifest.
2. `grep` of `internal/runproc` for kill and grace behaviour, plus `sed` of `internal/shedverbs/doc.go` lines 222-232. I needed this to check that a keeper which catches SIGTERM is still ended by `runproc.End`, and to quote the `OnStart` clause.
