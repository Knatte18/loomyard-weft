# Resume a writer whose strand is gone; stop a dead step's shuttle runs

Two failures where a strand or shuttle run left behind by an earlier interruption stopped a run.

## A gate re-prompt to a gone strand resumes the writer's session

A gated wait whose writer had to be re-prompted ended at once when the re-prompt found the strand gone:
reed no longer tracked it, held no pane for it, or its pane had died.
The run then landed as Stuck with "gate failed its first check and the session was never re-prompted", and a human had to pick up the findings file.

`Run.handleGatedBoundary` now resumes the writer's session in a fresh strand when the re-prompt fails that way (`relaunchForReprompt`, in the new `internal/shuttleengine/relaunch.go`).
The engine prepares a resumed launch of the run's session in the same run directory, with the re-prompt as the session's first turn, and the run's record names the new strand before its startup is awaited.
The run keeps its run directory, events file and gate state, and the resume counts as a delivered re-prompt.
The liveness lookup's three "can take no keys" answers now wrap one sentinel, `errStrandGone`, with their messages unchanged, so the gate path can tell them from a busy session.
`strandAddSpec` is the one place a strand's reed add request is built, shared by the first launch and the resume.

The resume runs only when the engine can resume a session (`SessionResumer`) and name the process holding one (`SessionHolderFinder`), and no live process holds the session.
The package has always refused to respawn over an untracked strand, because reed's bookkeeping can be reset under an agent that is still working;
the holder check keeps that guarantee, so a writer working in a pane reed lost is never run twice.
When the resume is refused or fails before its strand registers, the loop ends as before, and `GateOutcome.EndedEarly` names both the gone strand and why the session was not resumed.

## A dead step's shuttle runs are stopped before the re-step

A `--until-stop` loop that died without an envelope killed what was left of its in-flight step, but left that step's shuttle runs reading `running` with no process waiting on them.
Following the printed way forward, the operator's re-step then met that leftover run (`a run reads running but its waiting process is gone`) and halted again at the same row.
Only the watchdog kill stopped a dead step's runs.

`LoopSpec` gains `OnStepOrphaned`, called with the step's trace id wherever a step's process is gone without an envelope and the watchdog did not kill it:
the dead-loop stop's in-flight child, a child that exited without an envelope, and the in-flight child of a recovered loop panic.
The note it returns ends the interrupted cause or `loop.detail`.
loom wires it to the same step-run stop the watchdog kill uses; `stepRunsNote` now renders the note for both hooks.
Because the dead-loop stop runs before the waiter reports the halt, the leftover is settled before any re-step.

`Runner.StopStepRuns` used to skip a run of the step whose strand reed reports dead or no longer tracks.
After a machine stop that is every run of the step, so the record stayed `running`.
It now stops every running run of the step through the stop verb:
the strand is removed when it is removable, and otherwise only the record is set to stopped.
A run whose output files all exist keeps reading `running`, as the stop verb always does, so `Attach` still harvests finished work.
A run with no step is still listed as untied only when its strand is removable.

## Design decisions

- The wait between the gate's first failure and the halt came from the re-prompt held busy, re-sent each tick until the run deadline.
  The re-prompt path already fails at once when the strand is gone, so this change resumes the session rather than adding another fast failure.
- The cause of the original untracked strand could not be fixed from the evidence left: the halt notes are gone.
  Every path that drops a strand from reed's table needs either a dead agent or an explicit removal,
  and reed's resume no longer drops a gated strand whose outputs exist.
  So the change recovers from the gone strand whatever removed it.
- A strand that goes untracked while the writer is mid-turn, not at the boundary, still ends through the liveness check with "the session ended" as the reason.
  Only the boundary re-prompt resumes, since that is the point where the run holds the text the writer must receive.
- The interrupted-step record names the producer the status file holds once the step ended.
  After `Code-Bouncer` bounces back, that is `Code-Burler-Review` while the history's last entry is still `Code-Bouncer`;
  the record and the history agree, so this is unchanged.
- A clean stop of a loop already records its cause: a terminating signal leaves the loop's own mark in the loop log, and the keeper marks the exit.
  A loop exit with no mark at all is a `SIGKILL` or a machine stop, which no process survives to record, so no new mark is added.
