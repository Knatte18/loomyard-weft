# lym adopt runs in the foreground over pushed state

`lym adopt <subject>...` now runs every step in the caller's process and prints each step as it ends.
It starts no detached worker and writes no adopt record.
Each step reads tracked state, so a re-run goes on from where the state stands.
The operator's rule is that nothing happens in the hub until the adopt is done and every failing test it found is fixed.

## What changes

The steps, in order:

1. One Bolt commit writes an `adopt` hold, this machine as the subjects' owner and this machine's registry entry, and pushes it.
   A hold another machine wrote refuses the adopt, naming that machine and the hold's time.
   A rejected push re-reads Bolt and refuses on an owner of these subjects or a hold another machine wrote since; otherwise it replays and pushes.
2. For each moved run: fetch its branches, archive local tips origin lacks, reset them to origin's, and add the pair from origin when this machine has none.
3. Write the takeover entry on each moved run's records branches and push it, then fast-forward the local branches to origin's tips.
   This drops this machine's hand-over marks of each moved slug.
4. Run the catch-up over the whole hub.
5. Deploy.
6. Regenerate the hub workspace file.
7. Run the hub's `adopt` gate in each subject's worktree, with its output on the terminal.
8. A green check clears the hold, drops this machine's `bolt-hold` pause entries and runs `orchcli.BeamUp` for each subject.
   A failing check files a fix task through the fix loop, replaces the hold with an `adopt-fix` hold naming it, runs `orchcli.BeamUp` and exits non-zero naming the task.

Every adopt re-reads its own hold from Bolt before each step from step 2 on.
It stops once the hold is gone or names another machine, naming the subjects' owner and starting nothing.

`lym adopt --take-hold <subject>...` is new.
It takes over a hold another machine wrote, rewriting it to name this machine and keeping its kind, fix task, try counts and the owners before the adopt, then runs steps 2 to 8.
A hold for any other set of subjects, an overlapping one included, is refused, naming the hold's subjects.
A shell-guard row in `internal/shellguard` denies the flag to agents.
The row's steer and notice say only that taking over another machine's hold is the operator's, and neither spells the flag.

A hold this machine wrote for the same subjects stays as it stands, kind, fix task and try counts included, and the adopt goes on from step 2.
A hold this machine wrote for any other set of subjects refuses a plain adopt, naming the hold's subjects.
Under an `adopt-fix` hold, a failing check files no second task while the fix task stands.

Other readers move from the adopt record to the Bolt hold:

- `lym beam status` shows the hold's kind, machine, time and subjects, the fix task and try counts under `adopt-fix`, and the `lym adopt <subjects>` re-run for this machine's own hold.
- The dobby's handover and the orch watcher read the hold.
- `lym adopt check`, run by the fix task's landing, reads the `adopt-fix` hold's try counts; on green it clears the hold and runs `orchcli.BeamUp`.

`--here` keeps its fast outcomes `no_subject` and `owned_here`, and writes no hold.
`--archives`, `--no-orch` and `--all` are kept.

Retired: `lym adopt worker`, the adopt record and `internal/adoptrecord`, the held-back project verdicts, the adopt pause step, the writers of the `adopt` and `adopt-hold` callers, and `--keep-local`, `--drop-local` and `--discard`.
`internal/adoptcli`'s package doc is rewritten to the foreground flow, and ways forward no longer name the retired flags.

## Design decisions

- `boardengine` gets one pull-first Bolt write that records the adopt hold (or, under `--take-hold`, the rewritten hold), the subjects' owner and this machine's registry entry in one commit.
  Its hold setters rewrite a hold to another machine, replace it with an `adopt-fix` hold, update try counts and clear it.
  Bolt replay in `replay.go` carries a commit's holds and the registry, and refuses an adopt commit over another machine's new hold.
- `fabricengine` holds the takeover entry's production writer with its rejected-push retry (`takeoverwrite.go`):
  a push rejected over the old owner's later transition writes the entry onto origin's tip, so the adopter reads the run at that transition.
- `battencli.HubRunSlugs` gives the hub-wide run list the catch-up needs.
- Adopt works from pushed state alone.
  The old owner's unpushed work reaches the adopter only through the archive tags that `--archives` lists.
- The shell-guard row is a guardrail over static text, so a written script or a quoted command substitution passes it.
  A bypass is bounded: no check is skipped, no run goes on before a green check, and the old adopter stops at its next hold re-read.
- `PATTERN-dobby-runs` is updated to say an adopt holds the hub through Bolt's holds until its check is green;
  `PATTERN-daemon-wakeups` and `docs/overview.md` are adjusted to match.

## To check by hand

No regression check ran for this task, and no integration suite section was recorded.

- `takeover-teardown-lock-refusal-unreachable` (design, MEDIUM): in the takeover's teardown, check that the lock refusal's way forward can be reached.
  The plan-stage review's fixer reported on it, and no fresh reviewer has seen the result.
- `fixhold-way-forward-names-only-failing-subject` (design, MEDIUM): check that `FixHold`'s lost-hold way forward in `internal/boardengine` names the hold's own subjects, not only the failing one.
  A later change addressed it; confirm it.
- Run two adopts at once from two machines against one origin and confirm one owner results and the other refuses naming the winner.
- Kill an adopt after its owner write and re-run it; confirm it ends as an uninterrupted adopt does.
- Confirm a green check leaves no `bolt-hold` pause entry on the adopting machine.
