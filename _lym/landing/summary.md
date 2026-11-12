# Discussion-Review round asks the parent orch and keeps its findings

A Discussion-Burler-Review round can now ask the parent orch to review the decision record alongside the chair's lens forks and any assistant review.
The chair keeps every parent finding, and Discussion-Burler-Fix fixes them in the same round.
No row, pause or extra round is added; the Bouncer still judges the round and approval stays its call.

## What changes

- New `loom.yaml` key `discussion_review_ask_parent`, default `false`, category `none`.
  A task's `run_settings` overrides the hub's value in either direction.
- A round that asks sends the parent one creel line opening `[discussion-review request]`.
  It names the slug, round, ask number, decision record path, deadline and the answer verb with its `--ask` flag.
  The line is signed `<shortname>:LOOM` whatever strand the review row's process carries.
- The ask lives in a never-tracked ask record under the run's `.lym` directory, so an archive or a clear does not remove it.
  The record holds the run's latest ask: number, seed, round, time, deadline, state and destination strand.
- The review step gets a parent consolidation stencil (`burler-parent-consolidation.md`).
  The chair waits with the await verb, then keeps every item of a `done` report as a finding tagged `origin: parent`.
  A missing severity becomes `BLOCKING`, a `decision` class becomes `design`, and a parent finding is never moved to `## Rejected`.
  Each such finding lists the report ids it covers in a new `parent_ids` key, which `burlerengine`'s `Finding` reads.
- A parent coverage entry joins the review gate after the review-parse entry.
  It closes a still-open ask, then fails naming every report id that no `origin: parent` finding lists.
  It never passes on its cap.
- A `done` report is copied beside the review once the review is accepted, as the assistant's copy is.
- `contracts/orchrole/orch-role.md` gains a section for `[discussion-review request]` mail.
  It tells the orch to hand the review to a one-shot fork that may run `lym loom decision add <slug> --by parent` and `lym loom parent-review answer`.
  It also names the assistant keys as the way to add an agent review.
- `orchcli` gains a post variant that takes a resolved strand and signs as the program `LOOM` alone.

The agent fan the request asks for already exists as `discussion_assist_reviewer_model` and `discussion_assist_reviewer_fan`, so no key or code is added for it.

## Design decisions to know

- **When a round asks.**
  The ask record stores the round-1 focus file's modification time at full precision and compares it exactly.
  A fresh start, a new generation and a clear each write a new seed; a goto, a resume and a restarted row keep it.
  A round asks when:
  - the run has no ask yet, or the seed changed;
  - `Discussion-Write` ran after the latest ask, so a record rewritten after a goto reaches the parent;
  - the latest ask, from the round before, ended `done` with findings;
  - the latest ask was `not-sent` in an earlier round.
  A record changed only by `lym loom decision add` or `replace` sends no new ask.
- **An ask already written is finished whatever the key now reads.**
  A restarted row posts a `pending-post` ask again to the stored strand, accepting a possible second mail.
  Any later state posts nothing and keeps the first deadline.
- **No parent is not an error.**
  A destination with no orch strand, or a post error, sets `no-parent`, logs it, and the round goes on.
  A dead orch the post still reaches leaves the ask `open` until the deadline sets `timeout`.
- **Departure from issue #87's "Plan starts only on approval".**
  When the Bouncer approves after Burler-Fix fixed the parent's findings, no later round runs and the parent does not confirm the fix.
  `discussion_max_bounces` of `0`, `discussion_auto_accept`, a hard cap or `lym loom circling accept` can end the perch with parent findings open.
- **The fork's limit is instruction text only.**
  `lym loom` verbs carry no fork check, and an in-session fork shares the orch's strand name, so it passes the caller guards.
  Its restriction to two verbs and one slug lives in `orch-role.md`, not in code.

## To check

- `seed-precision-untested` (scope, MEDIUM): the exact-match seed comparison at full file-system precision has no test that exercises the precision itself.
  Tests skip the nanosecond cases on coarse file systems.
- Not run by hand: a live round against a real orch, including the fork answering through `lym loom parent-review answer`.
