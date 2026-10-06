# Bugfix: session lifecycle, notices and run halts

`bugfix-sessions` · Task · bug, shuttle, reed, loom, orch, burler · running · Loom-Preflight

Depends on: [`skills-in-one-turn`](design-skills-in-one-turn.md), [`test-suite-prune-core`](design-test-suite-prune-core.md)

Bugfix only (operator, 2026-10-06): fix what each section below reports, with a regression test per fix that drives the real CLI wiring, not only fakes.
No new verbs, config keys or features; a reviewer treats any extension as a scope finding.
The board adds more bug sections here until the task starts.

Scope limits: from `orch-resume-without-attach`, only the terminal handoff, and the orch resume verb is not part of this task.
Operator decision 2026-10-06: `lyx loom start` and `lyx orch start` never attach or switch a tmux client, for any caller; attaching to reed is only `lyx reed attach`.
The attach code path and the `--no-attach` flag go (the flag may stay as an accepted no-op for one release if callers pass it); docs and stencils that mention the handoff follow.

## producer-session-handoff

Imported from [issue #395](https://github.com/Knatte18/loomyard/issues/395).

reed's `SendText` sometimes types a prompt into a Claude Code pane without submitting it, and the session sits with the text in its input box until someone presses Enter.

Seen 2026-10-05 at `lyx orch start --handoff …`: the launch pointer (`Read …/prompt.md in full first; …`, two wrapped lines) stayed unsent; the operator pressed Enter by hand.
The short skill loads before it (`/scribe:prose`, `/ly:board`) were submitted.

`SendText` (`internal/reedengine/io.go`) runs `send-keys -l <text>` and then `send-keys Enter` back to back.
Claude Code's input treats a fast burst of characters as a paste, and an Enter that arrives inside the burst becomes part of it instead of a submit.
The longer the text and the busier the machine, the likelier the race.

Fix options:

1. Send the text as a bracketed paste (`load-buffer` + `paste-buffer -p`), so the TUI knows where the paste ends, then Enter.
2. After Enter, confirm the input box emptied (capture the pane, or wait for the transcript's user record) and resend Enter once if not.

Option 2 also covers other causes of a lost Enter; the two combine.
Every typed delivery (shuttle launch pointers and skill loads, orch watcher reloads, batten notices, `SendMessage`-style relays through reed) goes through this path.


### From issue #399

Imported from [issue #399](https://github.com/Knatte18/loomyard/issues/399).

`reflectHalt` (`internal/loomcli/halt.go`) writes `loom-halt.md` and at once runs `reflectFriction(false)`, which spawns a reflection agent session on every halt.
That includes a `blocked` halt classified `escalation-to-human`, where the producer's own session is still live and holds the open question.

Seen 2026-10-06 on `test-suite-prune-core`: Plan-Write halted `blocked` with "agent is asking a question".
While the plan session stood waiting, the run spawned a friction-reflection session; its report said only "an escalation, a process signal and not a lyx defect".
To the operator this looked like the run moving on while its producer hung.

Expected: an `escalation-to-human` halt writes the halt note but spawns no reflection.
The terminal Friction-Reflect row reads the note anyway.


### From issue #400

Imported from [issue #400](https://github.com/Knatte18/loomyard/issues/400).

When a halted run is moved back to a producer with `lyx loom goto --to <producer>` and resumed, the producer spawns a fresh session (`ly:<slug>:plan-2`) but leaves the halted attempt's session (`ly:<slug>:plan`) alive.
Both sessions then hold the same producer's instructions, and anything that wakes the old one (an operator message, a submitted input) puts two agents on the same output files.

Seen 2026-10-06 on `test-suite-prune-core`: Plan-Write halted `blocked` because its prompt pointer was never submitted (#395) and a hub message ended a turn.
After `goto --to Plan-Write` and a re-step, `plan-2` started while `plan` still held the unsubmitted pointer; submitting it ran two plan writers at once until `plan` was removed with `lyx reed remove`.

Expected: re-invoking a producer after a halt removes that producer's previous strand before spawning the new one, or reattaches to it, never both alive.

### Webster recovery strand left behind (2026-10-06)

A failed `recover-batch` leaves its `ly:<slug>:recovery` strand alive and idle after reporting.
On `skills-in-one-turn` it stayed beside the new Webster Master after a `webster reset --to start` and `goto --to Webster`, until removed by hand.
The same rule applies: a finished or superseded session is removed, never left beside its successor.

## false-crash-resume

Imported from [issue #396](https://github.com/Knatte18/loomyard/issues/396).

### What happened

`lyx shed step` (the `step` verb) wrote a `loom-crash-resume` note ("loom resumed after a suspected driver crash at Webster-Bouncer") with `state: running`, `current_producer: Webster-Bouncer`.
The run history's last entry was `Webster / done`, recorded about six seconds before the step started.
The step then ran normally: a bouncer-seed shuttle run finished with `outcome=done`, and `shed: step done` reported `outcome=stuck next=Webster-Burler`.
Nothing in the trace shows a crash; the run was advancing from one producer to the next.

### Why it is abnormal

A crash-resume note is meant to mark a run that was last stopped without a clean handoff.
Here the previous producer completed and the next one had not yet started, so the note is most likely a false positive from the gap between two producers (the next row marked current, its run not yet begun).
The note also gives no reason for the "suspected" crash, so the operator cannot tell whether it is real.

### What to do

Check what `lyx shed step` / `lyx loom step` uses to decide that the previous driver crashed, and whether a run that is `running` with a freshly completed previous row should count.
Include the evidence (heartbeat, pid or lock state) in the note so a real crash and a false positive can be told apart.

Provenance: task `test-suite-measure`; notes: `loom-crash-resume.md`.


### From issue #392

Imported from [issue #392](https://github.com/Knatte18/loomyard/issues/392).

### What happened

`lyx shed step` wrote a `loom-crash-resume` friction note ("loom resumed after a suspected driver crash at Webster-Bouncer") for a run that had not crashed.

- The run's last history entry was `Webster / done`, recorded six seconds before the flagged step began.
- A `lyx board sync` ran in that gap, as it does after a finished producer.
- The flagged step itself ran normally: `shuttle: run started` then `run finished outcome=done`, then `shed: step done producer=Webster-Bouncer outcome=stuck state=running next=Webster-Burler`.
- The status was `state: running` with `current_producer: Webster-Bouncer`, which is the ordinary state between two steps.

### Why it is abnormal

A clean handoff between two steps was classified as "last stopped without a clean handoff".
The note sends the reader looking for a driver crash that did not occur.

Two smaller defects in the same note:

- It reports `history_entries: 14` but lists only five history lines, with no marker that the list is truncated.
- Its history ends at `Webster / done` and omits the later entries, so it cannot be used to tell whether the stop was clean.

### What to do

- Check the crash-detection condition in the `lyx shed step` / `lyx loom step` entry path: a run whose last producer finished `done` and whose next producer has not started should not count as crashed.
- State in the note when the history list is truncated, or list all entries.

### Provenance

Task slug `board-config-per-worktree`; note file `loom-crash-resume.md`.

## orch-watcher-stale-reload

Imported from [issue #390](https://github.com/Knatte18/loomyard/issues/390).

On 2026-10-05 the hub restarted a stale `lyx orch watch` (it ran a binary from 2026-10-02) with `lyx orch start --no-attach`, keeping the live orch session.
The new watcher at once ran the post-compaction reload: it typed `/scribe:prose`, `/scribe:conversation` and the role-file pointer into a session that had been working normally for hours.

The compaction it reacted to was a manual `/compact` hours before the restart.
On its first tick the watcher reads the events from the persisted cursor (`phase_events_offset`, last advanced on 2026-10-04 by the old watcher), finds that compaction through `CompactedSince`, and starts `startAutoReload` as if it had just happened.

Expected:

- A reload follows only a compaction newer than the session's last completed turn; a compaction followed by later turns needs none.
- The reload types only what a compaction loses. Claude Code re-injects the skills a session has invoked after a compaction, so typing them again is a duplicate; the role file, read through a pointer, is what is lost.


### From issue #393

Imported from [issue #393](https://github.com/Knatte18/loomyard/issues/393).

The orch session cannot load `ly:board`, though the role procedure (`.lyx/orch/role.md`) and the orch watcher's reload both point to it.

Seen 2026-10-05 in the hub's orch session, after deploying `c140db21a` with `update-plugins.sh`:

- `~/.claude/plugins/cache/loomyard/ly/1.0.0/skills/board/SKILL.md` exists, and the prime's `.claude/settings.json` enables `ly@loomyard`.
- The Skill tool answers `Unknown skill: ly:board`, both before and after the watcher's role reload.
- The session's Claude Code process started on 2026-10-03, and `lyx orch` keeps it alive across context cycles through `/compact` and a typed reload.

Claude Code reads the skill list when the process starts.
A skill added or renamed by a later deploy stays unknown to a long-lived session, and so does every procedure that names it.

Possible fixes:

1. A context cycle, or the first cycle after a deploy, restarts the Claude Code process (`claude --resume` or a fresh session with the orch note) instead of compacting in place.
2. `update-plugins.sh` tells the running orch that skills changed, and the watcher schedules a restart.
3. The reload message checks that each skill it names loads, and reports a missing one to the operator instead of failing silently in the session.

### Restart race (2026-10-06)

`kill <pid of lyx orch watch>` followed at once by `lyx orch start --no-attach` answered `attach-only` and spawned no watcher, since the old watcher still read as live.
A second `lyx orch start --no-attach` a moment later spawned it.
`start` should wait for a watcher it sees dying, or the restart needs its own verb.

## burler-yaml-frontmatter-repair

Imported from [issue #386](https://github.com/Knatte18/loomyard/issues/386).

A loom run halted as `failed` at the `Discussion-Burler` producer (burler adapter in `internal/shedadapters`).

Halt output:

```
shedadapters: Discussion-Burler (burler): round 3 attempt 3: run: burler: round reached done but its review file is invalid: burler: review file frontmatter is not valid YAML: yaml: line 12: did not find expected key (a summary containing a " character must be ONE double-quoted string covering the whole value, not a quoted fragment followed by trailing prose)
```

The reviewer agent wrote a review file whose frontmatter summary contained a quoted fragment followed by trailing prose, which is invalid YAML.
The error text already states the rule, yet the run still failed at round 3 attempt 3 with history_entries 8.
Possible fixes: have the burler round prompt show the quoting rule up front, feed the parse error back to the reviewer for a repair turn, or have the frontmatter reader tolerate or normalise an unquoted summary.

Provenance: task `test-suite-measure`, from note `loom-halt.md`.

## orch-resume-without-attach

Seen 2026-10-06: the prime's reed window jumped to other sessions "by itself" while the operator watched the orch.
The cause was the orch's own `lyx loom start` calls in task worktrees: the orch runs inside reed's tmux, so `$TMUX` is set, and `lyx loom start` without `--no-attach` runs `tmux switch-client` onto the task's session (`internal/loomcli/start.go`), taking the operator's terminal with it.
`lyx orch start` does the same without `--no-attach` (`internal/orchcli/handover.go`).

The orch role (`contracts/stencils/orch/orch-template-role.md`) tells the orch to resume a halted child with `lyx loom start`, an operator verb whose job includes handing over the terminal.
In practice the call also did not move the runs: a live driver resumed only once the orch messaged it.

### Direction

- The orch resumes a child through a verb with no interactive side effect: a resume verb that only records the decision and wakes the run's driver (or batten), never spawning or attaching anything.
  The role stencil names that verb and never `lyx loom start`.
- `lyx loom start` and `lyx orch start` never switch the tmux client when called from an agent (`LYX_STRAND_NAME` set), whatever the flags; the terminal handoff is for an operator's shell only.
