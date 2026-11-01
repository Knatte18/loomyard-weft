# Fixer report — discussion round 1

Every change is in `_lym/discussion/decision-record.md`.
F3 is `decision`-class and goes to the parent, so this report leaves it out.

## Changes

- **F1** (BLOCKING) — D6 now decides that `MultiLLMProducer` maps a writer seat's step end (`EndedStep` set) to `shedengine.Stuck`.
  The Stuck carries a non-empty pointer, the ended seat's first output, so the commit decorator commits `_lym/parts/`.
  Its reason names the seat, the part, and the gate findings path, the Accept error, or the outputs missing at resume.
  D6 cites `join`, `mapOutcome` and `planWrite.Call` as evidence.
  The D6 acceptance bullet now says a writer's exhausted gate or failed Accept leaves the run Stuck at `Parts-Plan-Write` with a clean tree and a reason naming the seat.
  The self-check narrowed the mapping (see below).
- **F2** (BLOCKING) — D8 refuses `join-parts` unless the current row is `Parts-Plan-Write`, read from the run status's current producer and the recipe's rows.
  The Bound now says the verb overwrites only the plan directory, only while that row is current.
  The D8 acceptance bullet adds the refusal while another row is current.
  The self-check rewrote the way forward (see below).
- **F4** (MEDIUM) — D6's Bound now says that with no live chair the probe stops every writer and the fresh path archives every part directory, so every part is written again.
  It cites `Resume` and `Call`.
  D5's resume rule now applies only to a resume with a live chair.
- **F5** (MEDIUM) — D5 now decides that a writer whose run ended done still counts as outstanding while its kept session reads mid-turn, citing `advisorWork.outstanding`.
  The D5 acceptance bullet adds that the chair's held turn sends the parent nothing while a done writer is mid-turn on a fix.
- **F6** (MEDIUM) — D3's shared spots now record the part exports they use.
  The parts check refuses a spot that uses an export no card position can satisfy, with a way forward to reorder the parts or move the dependency.
  D10's stencil bullet asks for the same field, and Open risks line 3 is restated.
  The self-check narrowed the refusal (see below).
- **F7** (LOW) — **fixed differently.**
  D6 binds each writer seat's `plan-part` gate to its part name.
  The review proposed keeping each seat's findings at `_lym/parts/<part>/` plus a fixed file name.
  I did not make that change, because the part directory is what the part check and `join-parts` parse as a plan directory, and a findings file there would sit among the cards.
  Each seat's findings instead go to the run's gate findings file under the key `Parts-Plan-Write-<part>`.
  That file is run state, which is where `Env.GateFindingsFile` (`internal/loomcli/wiring.go`, `shedrun.GateFindingsFile`) already keeps every row's findings.
- **F8** (LOW) — D1's rule now reads `Plan-Write`'s latest history entry, and the acceptance bullet adds the goto-recovered case.
- **F9** (LOW) — D3's parts check refuses a part name beginning with `archive-`, with the way forward to rename the part.
- **F10** (LOW) — D5 now says the resume calls Accept with a zero `shuttleengine.Result`, and that the part check reads only files on disk.
- **F11** (LOW) — The D8 acceptance bullet now limits the format-check claim to parts that use no shared-spot export.
- **F12** (NIT) — D7 now says `--part` is a sixth mode.

## Self-check

- fix-consistency, LOW: the Constraints line and D5's "advisor tables behave as before" did not scope the Stuck mapping. Fixed as proposed: the mapping now covers writer seats only, and the Constraints line says advisor tables keep today's outcome mapping.
- fix-consistency, LOW: D6 still said "post-Done commit decorator". Fixed as proposed: it now says the decorator commits on any non-empty pointer, on Done and on Stuck.
- fix-consistency, LOW: the D5 and D6 Rationales omitted the new engine changes. Fixed as proposed: each Rationale now names its change.
- fix-claims, MEDIUM: a resolved gate closure takes no part name. Fixed as proposed: the table source resolves the `part_gates` entry once per seat through a resolver that takes the part name.
- fix-claims, LOW: the part check also reads the parts file. Fixed as proposed: D5 says it reads the part directory and the parts file.
- fix-proposal, MEDIUM (F1): mapping every `EndedStep` to Stuck would halt the run on a transient start failure or death. Fixed as proposed: only a done writer with a failed gate or Accept error, or one missing at resume, maps to Stuck; a start failure or death stays an error that the run's transient classification can retry.
- fix-proposal, MEDIUM (F6): "at or after" refused a spot whose cards fit inside its first needing part. Fixed as proposed: the refusal covers only parts strictly after that part, and Open risks line 3 says placement inside one part stays the chair's call.
- fix-proposal, LOW (F2) and fix-way-forward, MEDIUM (D8 way forward), merged: the goto way forward could not be run by the callers it stops, and it would discard the reviewed plan. Fixed as proposed: the way forward now says the joined plan is edited in place past the row, and that a full redo is the operator's goto on a halted run, naming what that discards.
- fix-way-forward, MEDIUM: D1's way back, a goto onto `Plan-Write`, is refused from a run halted at `Parts-Plan-Write` because goto moves a run only back (`internal/loomcli/cli.go` goto help). Fixed with my own change, since the fork proposed none: D1's way back is now a rerun of the halted row, or, for a solo plan, turning `plan_split` off and going to `Drift-Baseline`.

## Disputed

None.

## Deferred

None.
