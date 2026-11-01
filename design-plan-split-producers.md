# Parts-Write and Parts-Plan-Write: the plan written in two steps, beside Plan-Write

`plan-split-producers` · Task · feature, loom, stencil, shed, profiles · running · Plan-Bouncer

## Seen

The Plan writer is the costliest session of a tapestry run: it reads the code for the whole task in one session, and its mean context grows with the task (334k on the latest landed runs, peaks to 576k), paid again on every turn and inherited by every twill lens fork.
See note `plan-writer-context` and `docs/research/batching-effect-audit-2026-11-01/report.md`.

## Design (operator, 2026-11-01)

Two new producers write the plan in two steps, as an alternative to today's `Plan-Write`, which stays as it is and stays the default.
The point is to bench them against `Plan-Write` afterwards (a research task of its own); this task builds them and benches nothing.

- `Parts-Write`: one agent reads the decision record and splits the task coarsely into parts, without going into detail.
  It writes one parts file: each part with a contract (what it takes in, the behaviour it delivers, the names it exports to other parts, the parts it depends on) and the shared spots no single part owns.
- `Parts-Plan-Write`: a seat table built when the producer starts, from the parts file.
  - One advisor seat per part, a fresh session, not a fork: it gets the decision record, the parts file and the name of its part, reads only the code its part needs, and writes cards in today's plan format for that part alone, to an output of its own.
  - The chair orchestrates: it reads the parts file and the parts' cards, not the code at large; it waits for every part, then joins them into one plan in today's format and place, numbering the cards, setting card dependencies across parts and settling the shared spots.
  - The joined plan passes the same `plan` gate as `Plan-Write`'s, so the Plan review and Webster that follow see an ordinary plan.
- The Plan review that follows reviews the whole plan as today; nothing below the plan changes.
- Discussion-Write is not touched: it stays the one producer that talks with the operator.
- No new loom recipe: the two producers sit in tapestry beside `Plan-Write`, off by default, chosen by a config flag in the profile, as side actions are (an ordinary shed producer with a config flag).

## Seat engine

`internal/seatengine` today runs a chair with advisors that only advise.
This task reshapes it so an advisor seat can be a writing assistant: a seat that writes a product of its own the chair joins, with the number of seats taken from data at start.
- The engine already lets an advisor write outputs the chair reads and lets a table builder decide the seats; what the advisor role, its directive and the chair's directive say must change.
- `Parts-Plan-Write`'s chair gets a directive of its own; `seat-directive-chair` keeps serving today's tables.
- A part whose cards fail validation is caught before the chair joins (a required advisor with an accept hook, or the like).

## Stencils (operator, 2026-11-01)

The stencils of `Parts-Write`, the part writers and `Parts-Plan-Write`'s chair are the heart of this task and are reviewed with special care.
- Discussion settles each stencil's content with the operator before Plan.
- `Parts-Write` stays coarse: it decides parts and contracts, never cards, and states how fine a split is too fine.
- A part writer writes cards for its part only, never edits another part's, and raises a contract it finds wrong to the chair instead of improvising.
- The chair reads cards, not code, beyond what the shared spots need.
- A part writer reads only its own part and the shared spots it touches, never another part's code.

## Parts-Write reading depth (operator, 2026-11-01)

How much `Parts-Write` reads is its main knob, set by a value its stencil takes from config, so the bench can try more than one:
- light: the decision record, `relevant.md` and `lym quarry`'s symbols only;
- medium: light, plus the signatures and package docs of the packages the task touches;
- heavy, reading the code as Discussion did, is not offered: it approaches today's Plan writer.
Discussion-Write already read the code and wrote down what matters, so at either depth a `Parts-Write` that cannot split without reading code says so rather than reading on.

## The chair's work

The chair writes no part of its own; it joins the parts:
- it numbers every part's cards into one sequence;
- it sets a card's dependency on the card of another part that makes a name it uses, and checks that what a part delivers matches its contract;
- it writes the shared spots `Parts-Write` named (a registry, a config, `PATTERN.md`, shared docs) as cards of their own;
- it writes what today's plan format needs at the top, so the `plan` gate passes.

## Open

- The name and place of the parts file and of each part's cards.
- Whether `Parts-Write` gets a review of its own; the default is none, the whole-plan review catching what a bad split causes.
- How the config flag swaps `Plan-Write` for the two rows (`entry_gate` exists today for `Drift-Baseline`).
- Models: `Parts-Write` and the part writers on Opus as `Plan-Write` is; the chair possibly lighter.
- The chair as an agent, as above, or the shared spots as a part of their own written by a seat, with numbering and cross-part dependencies joined in Go from the names the contracts export, leaving the lead only to wait and check.
