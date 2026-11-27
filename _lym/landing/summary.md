# Study run stops, who settled them, and propose a settlement table

Adds `docs/research/autonomy/stop-classes-2026-11-27.md` and its line in `docs/research/README.md`.
No code changes.
The report gives the design of unattended runs its facts, read from run records, notices and orchestrator transcripts
of one machine from 2026-11-13 to 2026-11-27. Nothing was run or probed.

What it holds:

- A table of stop classes with count, example run, who settled each and how, and hold time.
  Of 131 unrequested halted states, the driver and its agents settled 67, the orchestrator 54,
  and the records of 10 do not say. 62 further stops left no halted state.
  Review escalations are the largest halted class (62).
- Each class marked mechanical or judgment, with what the judgment is.
- A proposed rule table: settled by the driver, settled by the orchestrator under today's autonomy
  or under a stricter overnight level, or a severe halt that waits for the operator.
  It names three questions the design still has to decide.
- The package where each class settled without an agent would be handled.
- The notices that fired for an agent that was waiting correctly, and what each should name instead.
  The quiet notice looks for neither an operator's approval at `Raddle-Write` nor a held turn end awaiting a reply.

## Design decisions

- The report states its limits: no records exist for five days of the window,
  the mail journal starts 11-21, counts sorted by hand from transcripts are marked "about",
  and autonomy was off for nearly the whole window.
- The study covers one machine's runs only.
- Operator messages are paraphrased, not quoted.

## Merge danger

**Door:** two-way
**Blast radius:** docs
