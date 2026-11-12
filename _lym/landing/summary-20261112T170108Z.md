# Discussion-Review asks the parent orch; the chair keeps its findings

A Discussion-Review round can now ask the run's parent orch to review the decision record beside the chair's lens forks and any assistant reviewer.
The parent answers with the `lym loom parent-review` verbs the previous change added.
The round's chair keeps every finding the parent reports, and a gate entry holds it to that.
The parent's findings join the round's review, Discussion-Burler-Fix fixes them, and the Discussion-Bouncer judges the round as before:
no row, pause, extra round or parent decision is added.

## What changes

- The Discussion-Burler-Review row gains a parent seat (`shedadapters.ParentReviewSeat`, wired by `loomcli`'s `discussionParentSeat`).
  Before the probe and before any spawn, the row finds the round's ask in the ask record by the generation's seed and the round.
  The seed is the round-1 focus file's write time, in RFC 3339 with nanoseconds, now stored on each ask.
  - A round that already holds an ask posts it again only while it is `pending-post`, and otherwise sends nothing and keeps the first deadline.
    This holds whatever `discussion_review_ask_parent` now reads, so a key turned off before a restarted row still ends the round's ask.
  - A round with no ask sends one only with the key on, and only when the run has no ask of this seed,
    the status history records a Discussion-Write call after the latest ask,
    the round before ended `done` with findings, or an earlier round's ask was `not-sent`.
    `lym loom decision add` and `replace` send no new ask.
  - The row resolves the destination by `NotifyParent`'s routing once, records the ask `pending-post` with that strand and its capped deadline,
    posts one line, then sets it `open`.
    The line opens `[discussion-review request]` and names the slug, round, ask number, the decision record's path, the deadline as a wall-clock time,
    and `lym loom parent-review answer <slug> --ask <n>`.
  - No room under the chair's timeout records the ask `not-sent` with the cap's reason.
    An unresolved parent, a parent with no orch strand, and a failed post record it `no-parent`.
    The round goes on in every case.
  - An accepted review settles the ask, as does the hand-on of a review accepted before a restart:
    an ask still open is set `timeout`, and a `done` report is copied beside the review as `<review stem>.parent.yaml`, once.
- `orchcli` gains `ParentStrand` and `PostAsProgram`.
  The request posts through the latter, signed as `<shortname>:LOOM` whatever strand the row's process carries.
- `burlerengine`:
  - `RunOpts.ParentAsk` names the round's ask.
    The chair's review step then renders a new part, `burler-parent-consolidation`, after the cluster rules and any assistant part.
    It tells the chair to wait with `lym loom parent-review await`, keep every item of a `done` report as a finding tagged `origin: parent`,
    take the item's severity or `BLOCKING`, map `decision` or no class to `design`, list the covered ids in `parent_ids`,
    keep `origin: parent` on a merged finding and never reject one.
    On any other ending the chair writes one line saying the parent review did not arrive and why.
  - `Finding` reads the optional `parent_ids` key, and the review format stencil names it.
  - The review table's gate gains `ParentCoverageGateEntry` after the review-parse entry, in probed and resumed tables too.
    Under the ask record's lock it first sets an ask still `pending-post` or `open` to `timeout`;
    then, for a `done` ask with findings, it fails naming every report id no `origin: parent` finding covers.
    It never passes on its cap, and a done chair whose gate failed with no reason now fails with the entry's findings, so the error names the missing ids.
  - The ask record gains `RoundAsk`, `EndParentAsk`, `SettleParentAsk`, `ReadParentReport` and `ParentCopyPath`.
- `contracts/orchrole/orch-role.md` gains "Reviewing a decision record" for the request line.
  It hands the review to a one-shot fork that may run `lym loom decision add <slug> --by parent` and `lym loom parent-review answer` for the named slug only,
  as an exception to the forks-only-read rule.
  It gives the findings file's shape and names the D1 assistant keys and the two parent keys with their `run_settings` override.
- Package docs and `docs/overview.md` describe the seat.

## Design decisions

- The ask step runs before the probe of a live reviewer, so a resumed table carries the same coverage entry as the spawned one.
  A reviewer live for a round always followed its round's ask, so the step sends nothing new there.
- `discussion_review_ask_parent` is read through the wiring's setting lookup at each round.
  An unreadable or non-boolean value is logged and read as `false`.
- The capped wait is computed only when a round sends a new ask, so its Warn is logged once per ask rather than at every wiring.
- A record or history read failure is logged and the round goes on without the parent, since the parent seat never fails a round.
- The chair's await runs without a slug: the chair runs in the task worktree, which the verb addresses itself.
- D1's assistant reviewer needed no code: the existing `discussion_assist_reviewer_model` and `_fan` keys already add an agent review, and the orch-role section names them.

## To check by hand

- No live round against a real orch was run; the tests cover the asking rule, the post, the settle, the coverage entry and the wiring's readers.
- Read the new orch-role section and the parent consolidation stencil as the orch and the chair will.
