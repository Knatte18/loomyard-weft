# Hardener: the hardener recipe, its round rows, sandbox hubs and live gate, with treadle and crucible retired

`hardener-campaign` · Task · feature, shed, loom, burler, gate, batten, report, board, config, landing, stencil, preflight, pattern, docs · running · Preflight

Decision record: [Sketch](decision-hardener-campaign.md)

Depends on: [`parent-review-code-and-drift`](design-parent-review-code-and-drift.md)

Filed from the Sketch `hardener`.

Takes: the Sketch's decision record whole; the darn recipe and loomcli/darnwiring.go as the shape to copy; the closed recipe lists in shedrun/seed.go, shedcli/table.go, battencli/wire.go childRecipeSupported, loomrecipe (names.go, finalizerow.go, bouncerrows.go), reportengine (runs.go, timingspans.go) and boardengine/task.go; shedrecipe's BurlerFix entry and row gates (entries_burler.go, entries_gate.go), shedadapters' NewBouncer, settleUndecided and bouncerfacts.go; loomcli's loomSegmentPerches and reviewBudget; gatecli/run.go acquireSlot and gateslot.Pool.AcquireSlots; fabriccli.CloneAndWire, runproc.End, reedengine.ServerName and selfexec.LymBinary; single-repo-fabric-mode's landed pairteardown and loomcli wiring
Delivers: a board task with recipe hardener runs a Preflight, Record-Write and Harden-Review ring campaign of Harden-Bouncer, Harden-Burler-Review and Harden-Burler-Fix rounds with a Go-run live gate, sandbox hubs reaped around every round row and on every teardown route, a per-round archive under _lym/harden/, and lands through Finalize; lym gate test --copies; the hardener.yaml config unit and the harden-burler-fix, harden-live and harden-live-copies gate template keys; treadle, Tenter and crucible gone, their method in the internal/hardenshed package doc and the harden-rubric-review stencil
Exports: the hardener recipe value; the HardenFix engine; lym harden sandbox new, list and clear; lym gate test --copies <n>; the hardener.yaml unit; the harden commit seam; the agents reinterpretation that strips treadle-judge and treadle-targeting

The part changes these packages:

- internal/hardenshed (about 3000 changed lines)
- internal/hardencli (about 400 changed lines)
- contracts/recipes (about 320 changed lines)
- internal/shedrecipe (about 250 changed lines)
- internal/shedbuild (about 200 changed lines)
- internal/shedadapters (about 500 changed lines)
- internal/shedrun (about 20 changed lines)
- internal/shedcli (about 15 changed lines)
- internal/battencli (about 15 changed lines)
- internal/loomrecipe (about 40 changed lines)
- internal/loomcli (about 700 changed lines)
- internal/loomengine (about 150 changed lines)
- internal/reportengine (about 30 changed lines)
- internal/boardengine (about 15 changed lines)
- internal/preflightshed (about 100 changed lines)
- internal/configreg (about 20 changed lines)
- internal/gatecli (about 500 changed lines)
- internal/gateslot (about 20 changed lines)
- internal/pairteardown (about 200 changed lines)
- internal/landingshed (about 80 changed lines)
- internal/agentconfig (about 200 changed lines)
- internal/treadleengine (about 5523 changed lines)
- contracts/stencils (about 700 changed lines)
- internal/verifyrun (about 20 changed lines)
- internal/webstercli (about 20 changed lines)
- cmd/lym (about 40 changed lines)
- crucible (about 1176 changed lines)
- docs (about 150 changed lines)
- pattern (about 20 changed lines)
