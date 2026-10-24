# A parent decision reaches the agent that needs it

Two routes kept a decision from the agent that had to act on it.
This change fixes both.

## Burler decision-class findings go to the parent

A Burler fixer could neither ask for nor record a `decision`-class finding, so each round raised the finding again and the fixer deferred it again.
The fix row now routes those findings outside the fix step:

- `BurlerHalfDeps.NotifyParent` (fed from the new `shedrecipe.Env.NotifyParent`) gives the fix row a way to reach the run's parent.
  Before a fresh fix starts, `BurlerFixProducer` reads the accepted review through `burlerengine.ParseCurrentReview` and posts one line listing each decision finding's id, summary and fix, the review path and the way forward, `lyx loom decision add`.
  The resume of a live fixer posts nothing.
  A review that does not parse, or a failed post, is logged at Warn and leaves the round's outcome unchanged.
- The row runs the fix with the new `burlerengine.RunOpts.DecisionsRouted`.
  `Engine.Fix` then counts decision findings as no work.
  A review holding only those is handled like an empty one: the gates run once, and when they pass the fixer is skipped, with a report saying the findings went to the parent.
  A spawned fixer's work line says the class is not its own, so it neither fixes, disputes nor defers those findings.
- Loom's wiring fills the notifier with `orchcli.NotifyPrime` as the `LOOM` program, prefixed with `loom <slug>:` so the orch can tell which run sent it.
  A row with no notifier, such as `lyx burler run`, behaves as before.
- The orch role stencil gains a bullet for this notice: decide each call and record it, or take it to the operator.

## The decision notice reaches Merriam

Merriam refused the `lyx loom decision add` notice for two reasons: it came from `ly:LOOM` rather than its parent, and it asked Merriam to read a record its stencil does not let it read.

- `decisionNoticeLine` now opens with a fixed phrase, `decisionNoticeOpening`, and carries the entry's decision and rationale inline beside the record path and heading.
- The parent directive's parent variant, which every spawned role renders, accepts a `[creel]` line from `<shortname>:LOOM` that opens with that phrase as a message from the parent.
  Every other session's message is still untrusted.
  A loomcli test checks that the rendered directive quotes `decisionNoticeOpening`.
- The webster Master stencil gains a section on parent decisions.
  The decision reaches the code through the forks Master spawns after it, and Master reads no record to get it.
  While its own work is still running, Master ends its turn without a tool call.
  When no batch is left, Master tells its parent that the decision can now reach the code only through a gate fixer or a plan amendment.
- The implementer and verify-fix bodies tell a fork that a parent decision in its context wins over its card or the plan where they differ.

Package docs for `burlerengine`, `shedadapters`, `parentdirective`, `websterengine` and the `loomcli` decision verb describe the new behavior.

## Design decisions

- The admission sits in the shared parent directive, not only in the Master stencil, so every role that receives the notice accepts it.
  Its bound: creel accepts a program claim only from an orch pane or a process in no pane, so no task agent can sign as `LOOM`.
- The notice still goes out as the `LOOM` program, not under the orch's strand.
  Creel types a program post into the pane in full, but delivers a strand post as mail behind a reminder, and the operator's shell has no strand.
- Decision findings are sent from the fix row and not through a Bouncer escalation, so the run keeps going while the parent decides.
  A call the parent leaves open is sent again in the next round that raises it.
- A fork that is already running when the notice arrives does not see the decision.
  The next fork Master spawns does.
