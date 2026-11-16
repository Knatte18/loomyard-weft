# Revise the remaining adopt-build members against what landed and what the operator decided since the split

`adopt-build-revision` · Task · research, docs, batten, dobby, orch, fabric, tender · running · Record-Write

Depends on: [`run-fsm-tracked-records`](design-run-fsm-tracked-records.md)

## Why (operator, 2026-11-15)

The beam `adopt-build` was split by raddle from the Sketch `adopt-redesign`, and the plan has changed a lot since:
the orch continues its own session on the same machine only, with no transcript mirror (F12);
the orch stays up during an adopt hold (F14);
`prime-is-root-beam`, `hub-stop-button` and `machine-restart-shortcut` joined the beam as darn tasks.
The two first members, `orch-resume-own-session` and `run-fsm-tracked-records`, land in the beam before this runs,
and what they built may differ from what the later members' bodies assume.

## Task

Revise every member of the beam that has not run, so each body and decision record matches what stands now:
`push-before-act-takeover`, `bolt-holds-catch-up`, `adopt-foreground-leftovers`, `prime-is-root-beam`, `hub-stop-button`, `machine-restart-shortcut`.

Read:

- the beam's branch as it stands after the two landings, and their decision records (`lym board get <slug> --record`);
- each remaining member's body and, where it has one, its decision record;
- the raddle split's record under Bolt's `raddle/adopt-redesign`.

For each remaining member, settle:

- what its `Takes` names that the landed members built differently, renamed or left out, and what it now takes instead;
- what the operator's later decisions above change in it, such as anything left from the transcript mirror or a cross-machine orch resume;
- overlap or gaps between members, the three darn members included: `hub-stop-button` builds on the run FSM and its pause and cancel intents;
- whether its `depends_on`, recipe and package list still hold, and whether two members should merge or one should split.

## Bounds

- A study: read code, records and the board; run no tests and change no production code.
- Change no board entry and no decision record: the orch applies the result after the operator approves it.
- The result is a report under `docs/research/` with one section per remaining member:
  what changes and why, the revised body in full ready to write to the board,
  and a revised decision record where the member has one and it changes.
  It ends with any change to order, dependencies or recipes, and the open questions for the operator.
