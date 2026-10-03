# Friction reaches GitHub on its own: reflection under step, no approval gate, notes kept until filed

`agent-self-reflect` · Tasks · feature · running · Webster-Bouncer

## Goal

Every run's friction reaches GitHub as issues, with no operator in the loop: agents write it down, a reflection files it through `lyx selfreport`, and nothing deletes it first.
Filtering happens when issues are pulled back into the board, not before filing (operator, 2026-10-03, as in millhouse).

## Today (seen 2026-10-03)

- Agents write friction notes to `_lyx/loom/friction/` (Webster forks, recovery strands, Burler rounds), and they are committed to weft, but nothing reads them in a driven run.
- Friction-Reflect reflects only under `lyx loom run` (`internal/loomcli/run.go`, `reflectFrictionRow`, `armedVerb == "run"`), because "the reflection agent files public issues itself".
  Every ly-drive driver uses `lyx shed step`, so it always skips.
- ly-drive's `## Self-report` drafts one `lyx selfreport create` per run, files it only on operator approval, and files nothing in autonomous mode; every orch-driven driver is autonomous, so the draft dies in the drive report.
- Batten's teardown deletes the friction notes with the pair (#274, note `batten-llm-friction`, folded in here).
- Everything worth filing on 2026-10-03 (#348–#352: Webster's Delete ordering, a failed batch that cannot be redone, plan_drifted on an already-deleted file) was found and filed by the hub by hand.

## Shape

- **No gate.** Friction-Reflect reflects under `step` as well as `run`; ly-drive files its self-report in every mode, or hands it to Friction-Reflect instead of drafting its own.
- **Sources.** The reflection also reads the driver's drive reports and repair records (ly-drive already calls repair records friction data), and Merriam writes a friction note whenever a lyx verb refuses it (`rebaseline`, `begin-batch`, `recover-batch`), so that friction exists in writing.
- **Self-reflection, if the sources are not enough.** Merriam and the driver reflect on their own session at the end of their work and file through `lyx selfreport`, as `mill-self-report` does in millhouse.
  Decide in Discussion whether the sources above already cover it.
- **Ordering.** The reflection finishes before batten tears the pair down, and the notes survive a failed reflection.
- **Filing.** Through `lyx selfreport create` only, never `gh`; dedupe against open issues, or leave that to the intake filter (decide in Discussion).

## From `convergence-plan`: Friction on halt (hub, 2026-10-02)

A refusal or halt makes a friction note mandatory, so every intervention leaves a record the measurement can count.
Check first whether `internal/loomengine/anomaly.go` loses a halt that is later resumed.
Tier 1 anomaly filing (`detectAndFileAnomalies`) also runs under `step`, since the no-gate decision covers every filing path (added after Discussion, 2026-10-03; the decision record had kept it `run`-only).

## Open

- Under `step`, the reflection blocks the driver's `shed step` call for up to `friction_timeout_min`; ly-drive must tolerate that wait without taking the run for hung.

## Done when

- A driven run that leaves friction notes, a refusal or a repair ends with matching issues on GitHub, filed without anyone approving them.
- Re-running the 2026-10-03 failures through a run would have filed #350 and #352 without the hub.
