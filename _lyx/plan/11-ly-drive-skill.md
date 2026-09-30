# Card 11 — ly-drive skill: safe re-steps, parking, binary watch, resume

**Prosa:**
- `plugins/ly/skills/ly-drive/SKILL.md`
- `docs/overview.md`

**Create:**
- `internal/loomcli/driveskill_test.go`

**Uses:**
- `plan:internal/shedrun#ParkMarkerFileName`

**Intent:**
Teach the driver the new behaviour (decision record D3, D5–D11) while it stays recipe-blind: every new branch is on an envelope or status field, a policy word or one of the five kinds, never on a row or recipe name, and the skill still makes no commits.

`SKILL.md` changes, each in the section that owns the topic:

- **Error envelopes**: the error envelope also carries `transient`, a class name or empty; the driver reads that key and never matches error text.
- **Automatic re-steps** (a new section, a closed list of exactly two, each with its own budget, separate from the repair cap):
  - *Transient re-step*: an error envelope with a non-empty `transient` gets one immediate re-step, with no repair action and no wait.
    Record `current_producer` from a status read first; if that re-step stops again at the same `current_producer`, for any reason, hand back.
  - *Binary-change re-step*: when `lyx shed status`'s `last_step.binary_changed` is true, re-step once.
    It is checked at the stop itself, and again each time the watch fires while parked.
    Eligible stops: `blocked`; a `producer`, `bootstrap` or `unseeded` error the driver escalated (repair cap exhausted or nothing to repair); a transient stop whose own re-step failed.
    Never eligible: `awaiting`, `paused`, `busy`, `ownership`, a no-kind refusal, and an interrupted invocation under `interrupt_policy: handback`.
    `awaiting` and `paused` are never auto-resumed, by either re-step.
  - *Clean-tree guard*: skip the binary-change re-step, and stay parked, when `git -C <warp> status --porcelain` in the task's code worktree lists any change outside `_lyx/` and `.lyx/`.
    The driver's own stop reports, repair records and park marker never count.
    A skipped re-step is not retried when the tree turns clean; it waits for the next binary change or a resume.
  - Before any self-initiated re-step, remove the park marker.
- **Parking** (a new section): only a session whose launch prompt says to park does so.
  Orchestrator forks and operator-launched sessions keep their stop behaviour and get the transient re-step and the stop-time binary-change re-step only.
  At a hand-back, a parking driver does four things, in order:
  1. writes its stop report;
  2. runs `lyx loom commit-records`;
  3. writes `<scratch_dir>/driver-parked` holding the stop report's path;
  4. starts the binary watch as a background job when the stop is binary-change eligible.
  It then ends its turn with the session open.
  At `done` and at `busy` it runs the launch prompt's end-of-session command instead, as today.
  On every wake (a resume line, a background job's exit, a context compaction), the marker on disk is the truth of whether the driver is parked.
- **Binary watch**: a background shell loop that resolves the `lyx` executable once, checks its modification time about once a minute, and exits when it changes.
  The watch never runs a long-lived `lyx` process, which would block a deploy's overwrite on Windows.
  When it exits, read `lyx shed status` once, and re-step only if `binary_changed` is true and the guard passes; otherwise re-arm the watch.
  A watch job that ends by the harness timeout is re-armed, never read as a resume.
- **Resume**: a resume line names a report path.
  On it, stop the driver's own watch job, remove the marker if still present, take a fresh baseline, reset the repair and re-step budgets, and write every later stop report of this attempt to that path.
  A resume line that arrives while a step job is in flight starts no second step; its report path and budget reset apply to the next stop.
- **Repair records and the stop report**: each automatic re-step writes a record under `<scratch_dir>/repairs/` holding:
  - the stop;
  - the field that justified it: the `transient` class, or the build identity `last_step` recorded before the re-step and the one it records after;
  - the action and the outcome.
  After a self-initiated re-step, the next stop rewrites the same report file to cover the whole attempt, listing every automatic re-step since the attempt began.
  The end-of-session paragraph distinguishes the teardown at `done`/`busy` from `lyx loom commit-records` at a park.
- **Repair cap**: states that the re-step budgets are separate from it.

`docs/overview.md`, in the shed module entry's ly-drive sentence and the loom module's `commit-records` sentence: the driver re-steps once on its own after a transient failure or a deployed `lyx`, and a loom-launched driver parks at a hand-back, running `commit-records` there, until `lyx loom start` resumes it.

The skill names the marker's filename exactly as `shedrun.ParkMarkerFileName` spells it, in backticks.

`driveskill_test.go` (untagged): locate `plugins/ly/skills/ly-drive/SKILL.md` from the test file's own directory via `runtime.Caller`, and assert the skill contains the backtick-quoted `shedrun.ParkMarkerFileName`, so renaming either side without the other fails.

**Commit:** 11: ly-drive: safe automatic re-steps, parking, binary watch and resume
