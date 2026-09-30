# ly-drive: safe self-repairs instead of always handing back

## Goal

A loom-launched ly-drive driver clears the stops that a single re-step would clear, without the hub.
It re-steps once on its own after an infrastructure failure lyx has marked transient, and once after a new lyx binary is deployed while it waits.
Every other stop is still handed back, and the driver no longer exits at a hand-back: it parks in its own session, and `lyx loom start` resumes it by typing one line into that session instead of spawning a fresh driver.

## Scope

In:

- A transient mark on producer and bootstrap errors, declared once in Go, set only at a closed list of infrastructure-failure classes, persisted in the status file and carried on the step error envelope.
- A build identity (VCS revision) recorded for every `lyx shed step` invocation, and a derived `binary_changed` field in `lyx shed status`'s `last_step`.
- A park marker file in the run's ephemeral scratch directory, set by a parked driver and consumed by `lyx loom start`.
- `lyx loom start` resuming a parked, live driver through a shuttle `Send` of one resume line that carries a fresh drive-report path.
- The ly-drive launch prompt: a hand-back parks the session; only `done` (and `busy`) runs the existing end-of-session teardown command.
- The ly-drive skill: a closed list of safe re-steps, the parked state, the binary watch, the resume message, and repair records for every automatic re-step.
- Docs: the ly-drive `SKILL.md`, `lyx loom start` help text, `docs/overview.md`, and a new `CONSTRAINTS.md` invariant for the transient mark.

Out:

- Flaky or pre-existing verify failures. `verify-triage` handles them in Go (rerun, parent compare), so the driver gets no flaky-verify class.
- Auto-approving `awaiting`. `lyx loom approve` is unchanged and still never resumes a run.
- The Go driver (`lyx loom run`): it neither parks nor reads the transient mark.
- Orchestrator-fork and operator-launched ly-drive sessions do not park; they get the transient and stop-time binary re-steps only.
- The reed mailbox (`manifest/designs/reed-mailbox.md`). The resume channel is the existing shuttle `Send`.
- The `driver-restart-race` board note's own fix. Parking removes its trigger at hand-backs; the `done` teardown path is unchanged.
- Changes to the existing trace-based repair path, its repair cap, or the stranded-branch exception.

## Decisions

### D1 — Transient mark is Go-classified, never text-matched

Decision:
A typed transient mark, declared once in `internal/shedengine`, wraps a producer or bootstrap error whose cause is outside the task.
Shed persists it with the `failed` state, and the step error envelope carries it as a `transient` key holding the class name (empty when not transient).
The refusal-kind vocabulary stays at its five values; `transient` is a new envelope key, not a kind.
Rationale:
The skill branches only on envelope fields, policy words and the five kinds, never on text.
Only the code at the failure site knows whether a failure was a transport fault or a real rejection.

### D2 — Closed list of transient classes

Decision:
Three classes, and no others:

1. `git-transport` — a git remote operation (push, fetch, ls-remote, clone) that failed to reach the remote: connection refused or reset, DNS failure, timeout.
   A rejected push, a non-fast-forward, an auth failure or a missing ref is never transient.
2. `github-api` — a GitHub API call that failed in transport or with a 5xx response.
   A 4xx, including a rate limit, is never transient.
3. `agent-start` — an agent session whose provider never came up ready (`shuttleengine.ErrNotStarted`).

A gate verdict (`blocked`, `awaiting`, `paused`) is never marked.
Lower-level packages expose their own classification; the producer or bootstrap boundary translates it into shedengine's mark, so leaf packages gain no shedengine import.
Rationale:
These are the failures the body names ("a verb that failed on infrastructure") that a second attempt a moment later is likely to clear.
A closed list keeps the driver from re-stepping a real fault.

### D3 — One immediate re-step per transient stop

Decision:
An error envelope with a non-empty `transient` gets one immediate re-step, with no repair action and no wait.
If that re-step stops again at the same `current_producer` for any reason, the driver hands back.
This budget is separate from the existing two-repair cap on trace repairs.
Rationale:
One retry clears a blip; a failure that repeats at once is not a blip.

### D4 — Build identity per step, `binary_changed` derived by status

Decision:
Every `lyx shed step` records the VCS revision and modified flag of the binary that ran it in its step record.
`lyx shed status` reports that identity on `last_step` and derives `last_step.binary_changed`: true only when both the running binary's identity and the recorded one are known and differ.
An unstamped binary on either side yields false.
Rationale:
Production binaries are built from a clean pushed tree and carry `vcs.revision`, so "a deploy landed" is exactly "the revision changed".
Unknown identity yields no automatic re-step, which is the conservative failure.
Because the re-step itself records the new identity, `binary_changed` turns false after it, which bounds the re-step to once per deployed binary with no counter.

### D5 — Binary-change re-step: eligible stops and the clean-tree guard

Decision:
A driver re-steps once when `binary_changed` is true, checked immediately at the stop and again each time the binary changes while parked.
Eligible stops: `blocked`, a `producer`, `bootstrap` or `unseeded` error the driver escalated (repair cap exhausted or nothing to repair), and a transient stop whose own re-step failed.
Never eligible: `awaiting`, `paused`, `busy`, `ownership`, a no-kind refusal, and an interrupted invocation under `interrupt_policy: handback`.
The driver skips the re-step, and stays parked, when the worktree has any uncommitted change; `git status --porcelain` is the read-only check.
Rationale:
The hub's usual recovery was "fix lyx on `main`, deploy, `loom start`", and most 2026-09-30 hand-backs were of that kind.
`awaiting` and `paused` wait on a person by design.
An interrupted hand-back may still have a live agent that a re-step would restart.
A dirty worktree means someone is fixing the task by hand, and a re-step would run against a half-edited tree.

### D6 — Parking replaces exiting at a hand-back

Decision:
A loom-launched driver at a hand-back writes its stop report, runs `lyx loom commit-records`, writes the park marker, starts its binary watch as a background job, and ends its turn with its session still open.
At `done` and at `busy` it runs the existing end-of-session teardown command, as today.
Orchestrator forks and operator-launched sessions keep their current stop behaviour.
Rationale:
A resume then costs one typed line instead of a fresh session, and the driver keeps the context of what it already tried.
`done` must still end the session, because batten waits for the driver to exit before teardown.
`busy` means another driver holds the run, so this one has nothing to resume.

### D7 — Park marker in the run's ephemeral scratch directory

Decision:
The marker is a file under the run's ephemeral scratch directory (the envelope's `scratch_dir`), holding the path of the stop report it parked on.
Its filename is declared once in Go through `internal/shedrun`, and the skill names the same filename; a test pins the two together.
The driver removes it before any self-initiated re-step; `lyx loom start` removes it when it resumes the driver.
On any wake (a resume line, a background-job exit, a context compaction), the driver treats the marker on disk as the truth of whether it is parked.
Rationale:
`loom start` needs a deterministic "this live driver is waiting" signal: the pane alone cannot tell a parked driver from one waiting on a background step job.
The file is machine-local run state, so it belongs under `.lyx`, per the Durable-vs-Ephemeral State Invariant.
Reading state from disk on wake survives context compaction in a long-lived session.

### D8 — `lyx loom start` is the resume route

Decision:
`lyx loom start` branches on the driver strand and the marker:

- Live driver with the marker present: compose a fresh drive-report path, `Send` one resume line naming it, remove the marker, spawn nothing.
  With `--no-attach` it returns once the `Send` is verified; without it, it then attaches as today.
- Live driver without the marker: the driver is working; behave as today.
- Dead or absent driver: remove any stale marker, then spawn as today.

When the pane is not ready for input, `loom start` waits a bounded time, then refuses naming a retry and leaves the marker in place.
The resume line tells the driver to take a fresh baseline, reset its repair and re-step budgets, and write its next stop report to the named path.
Rationale:
The hub (`loom approve` then `loom start`) and batten (whose `Spawn` runs `loom start --no-attach`) keep their existing commands unchanged, and neither needs to know that a driver parked.
Typed input through shuttle's `Send` needs no approval in the driver's pane, unlike a cross-session message.
A per-attempt report path keeps Spec's no-existing-output-file rule and one report per attempt.

### D9 — Report files per attempt

Decision:
A spawned or `loom start`-resumed driver writes all its stop reports to that attempt's report path.
After a self-initiated re-step, its next stop rewrites the same file to cover the whole attempt, listing every automatic re-step.
Every hand-back runs `lyx loom commit-records`, so git keeps each version.
Rationale:
Only Go mints report paths.
Rewriting within one attempt loses nothing that is not already committed.

### D10 — Binary watch is a cheap file check, confirmed by one status read

Decision:
While parked on an eligible stop, the driver runs a background shell loop that checks the modification time of the resolved `lyx` executable at a fixed interval of about a minute and exits when it changes.
The driver then reads `lyx shed status` once and re-steps only if `binary_changed` is true and the D5 guard passes; otherwise it re-arms the watch.
A watch job that ends by the harness timeout is re-armed, not treated as a resume.
When a resume line arrives, the driver stops its own watch job before stepping.
Rationale:
Polling `lyx shed status` itself would write a trace file per poll for hours.
A long-running `lyx` wait process would hold the executable open, which blocks the deploy's overwrite on Windows.

### D11 — Every automatic re-step is a repair record

Decision:
Each transient or binary-change re-step writes a record under `<scratch_dir>/repairs/`: the stop, the field that justified it (`transient` class, or both build identities), the action and the outcome.
The next stop report lists every automatic re-step since the attempt began.
Rationale:
The body asks for records under `<scratch>/repairs/`; the stop report is the durable copy, since `scratch_dir` is machine-local.

### D12 — New CONSTRAINTS.md invariant: Transient Stop Invariant

Decision:
Record that the transient mark is declared only in `internal/shedengine`, that its class set is closed at the D2 list (stated once, at its declaration), that it is set only at a producer or bootstrap boundary, and that a gate verdict never carries it.
Rationale:
It is cross-cutting: any producer can set it, and a mark added at a new site changes what the driver retries without a human deciding.

## Constraints

- `CONSTRAINTS.md` governs; in particular:
  - Shed Producer-Seam Invariant: `internal/shedengine` imports only stdlib, `state` and `lock`, so the mark is a stdlib-only type there.
  - Shed Verb-Set Invariant: the step refusal-kind vocabulary stays at five; the envelope key sets are closed by doc comment and test, so both get updated with the new keys.
  - Shed Run-Directory Invariant and Lyxdirs Single-Declarer Invariant: the park-marker path is composed through `internal/shedrun`, never spelled in `loomcli`.
  - Durable-vs-Ephemeral State Invariant: the marker lives under `.lyx`.
  - Shuttle Provider-Seam Invariant: the resume line is prompt text; no Claude-specific flag or command line in `loomcli`.
  - Driver Choice Single-Site Invariant: resume versus spawn branches on strand liveness and the marker, not on a second read of the seed's driver field.
  - Live-Substrate Spawn Observability: the resume `Send` and any wait are logged.
  - CLI/Cobra Invariant and Documentation Lifecycle: help text and docs change in the same commit.
- `shuttleengine.Send` accepts a single non-empty line only.
- The launch prompt stays under `maxLaunchPromptBytes`.
- The skill stays recipe-blind: every new branch is on an envelope or status field, never on a row or recipe name.
- The skill still makes no commits; `lyx loom commit-records` commits the records.
- No long-running `lyx` process during a park (Windows deploy overwrite).

## Auto-mode assumptions

- A single re-step is the whole automatic budget per transient stop and per deployed binary; the operator asked for "one re-step".
- `blocked` is eligible for the binary-change re-step even though some blocks are real human decisions: the re-step is what the hub does after a deploy anyway, and it costs one producer call.
- A rate-limited GitHub call is not transient, since an immediate re-step hits the same limit.
- `unseeded` and `bootstrap` errors after the repair cap are eligible for the binary-change re-step, since a lyx bug can cause either.
- The watch interval is about a minute; a deploy is picked up within that.
- The hub can hold a parked driver during a hand fix with `lyx loom pause`: the next re-step then persists `paused`, which is never auto-resumed.
- Dev builds from a modified tree share a revision with their base, so they trigger no binary-change re-step; the dev route is for internal testing only.

## Open risks

- An unrelated deploy re-steps every parked, eligible run once; for an LLM producer that is one agent run per parked run.
- A parked driver's session grows over many resumes and may compact; D7's on-disk marker is the mitigation.
- A deploy landing between a hub subagent's commits in a task worktree passes the clean-tree guard and re-steps mid-fix, unless the hub paused the run first.
- A small race remains where `loom start` reads the marker just as the driver self-resumes; the skill rule that a resume line during an in-flight step job is noted and ignored covers it.
- Classifying git and GitHub failures by their error text inside lyx may misclassify a new message format; the fallback is "not transient", which is today's behaviour.
- The HANDOFF's "wait until the session has no `claude` pane left" step becomes unnecessary once this lands; updating that operator-owned file is the hub's call.

## Acceptance criteria

- A producer error in each D2 class persists `failed` with the class, and the step error envelope carries `transient: <class>`.
- A rejected push, an auth failure, a 4xx GitHub response, and every gate verdict carry an empty `transient`.
- The step error envelope's closed key-set test and doc comment include `transient`; the success envelope is unchanged.
- A step record carries the build identity of the binary that ran it.
- `lyx shed status` `last_step.binary_changed` is false for equal identities, true for differing known identities, and false when either identity is unknown.
- `lyx loom start` with a live driver and the marker present sends exactly one resume line naming a new drive-report path, removes the marker, and spawns nothing; with `--no-attach` it returns after the verified `Send`.
- `lyx loom start` with a live driver and no marker neither sends nor spawns.
- `lyx loom start` with a dead driver and a stale marker removes the marker and spawns a fresh driver.
- `lyx loom start` against a pane that never becomes ready refuses after the bounded wait, names a retry, and leaves the marker in place.
- The launch prompt tells the driver to park at a hand-back and to run the teardown command only at `done` or `busy`, and stays under `maxLaunchPromptBytes`.
- The park-marker filename in the skill matches the Go declaration, pinned by a test.
- `SKILL.md` states the closed list of automatic re-steps (D3, D5), their eligibility and guards, parking, the binary watch, resume handling, and the repair records; `awaiting` and `paused` are explicitly never auto-resumed.
- `lyx loom start --help`, `docs/overview.md` and `CONSTRAINTS.md` (Transient Stop Invariant) describe the new behaviour.
- The batten flow still works end to end: an awaiting child with a parked driver is resumed by batten's `Spawn` after `lyx loom approve`, and a `done` child's driver still exits before teardown.

## Notes for the plan writer

- Existing seams: `internal/loomcli/driverprompt.go` (prompt and `driverTeardownCommand`), `driverlaunch.go` (`driverStarter`, `driverPaneProbe`), `start.go` (`runDriverSpawnAndWait`, `startLLMDriverArm`), `driverreport.go` (per-attempt report path).
- `shuttleengine.Runner.Send(guid, text)` already reaches a run started by another process and verifies delivery.
- `internal/shedverbs/steprecord.go` holds the per-invocation step record `last_step` is read from; `stepErrFields` in `step.go` builds the error envelope.
- `internal/shedengine/run.go`'s `callErr != nil` branch persists `failed`.
- `runtime/debug.ReadBuildInfo` exposes `vcs.revision` and `vcs.modified`; the deployed binary carries both.
- Batten's child `Spawn` runs `lyx loom start --no-attach` (`internal/battencli/wire.go`).
