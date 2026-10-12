# Card 13 — shuttle-send-refusal

**Edit:**
- `internal/shuttlecli#shuttleCLI`
- `internal/shuttlecli#Command`
- `internal/shuttlecli#shuttleCLI.sendCmd`
- `internal/shuttlecli#TestSendCmd_AppendsMessageTail`
- `contracts/specs/refusal-spec.md`

**Intent:**
Refuse `lyx shuttle send` into a covered strand (D4), so lyx's own way of typing into an agent cannot wake a frozen one.
`shuttleCLI` gains `pause pauserecord.Geometry`, set by `Command`'s pre-run from `hubgeom.PauseGeometry(layout)` beside the runner (card 12).
`sendCmd` reads `pauserecord.ReadCoverage` for the geometry's slug before it sends; a covered pair refuses with `pauserecord.CoveredError(entry, slug, "re-run lyx shuttle send")` on the error envelope and types nothing.
The send verb is pair-scoped (its runner finds the guid under this worktree's anchor), so the pair's own coverage is the strand's.
An unreadable record does not refuse: it logs one Warn and sends, as the record's other readers fail open.

Reachability: a run started before the pause and a live entry for its slug (or a hub entry) reach the refusal after the argument checks pass; the guid need not exist, since coverage is checked first.

`refusal-spec.md` gains a `## pause` section (cards 15, 22 and 23 add to it) whose first row is this one: refusal `shuttle send under a pause`, trigger "`lyx shuttle send` names a strand of a pair a live pause entry covers", class transient, way forward "run `lyx resume <slug>`, or `lyx resume` for a hub-wide pause, then re-run the send".

Test: `TestSendCmd_AppendsMessageTail` gains a `covered` field and a row whose `shuttleCLI` carries a geometry over a temp record with a live entry for its slug: the envelope is `ok: false` with the error naming `lyx resume`, and the fake reed records no typed text; the existing rows carry an empty geometry and still type once.

**ImpactSummary:** One coverage read before the send verb's call; every other shuttle verb and every in-process send are unchanged.
