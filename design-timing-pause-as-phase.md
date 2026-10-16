# report timing counts a pause before Plan-Write as Plan phase time

`timing-pause-as-phase` · Note · bug, shed, batten

Imported from [issue #695](https://github.com/Knatte18/loomyard/issues/695).

Seen: lyx report timing --since 2026-10-14 gave twill-fork0 Plan 1h47m and kickstart-per-card Plan 1h30m. The driver transcripts show both runs were paused before Plan-Write on request (notices.log: 'twill-fork0: paused at Plan-Write; since 19:02:00Z', 'kickstart-per-card: paused at Plan-Write; since 10:11:04Z'), and the Plan-Write sessions ran only 11 and 13 minutes. lyx shed status also shows 'now: Plan-Write' during such a pause. Expected: time a run spends paused at a producer boundary is reported apart, like machine sleep and waits, not as the next phase's time.