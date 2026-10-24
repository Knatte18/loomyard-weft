# Route Burler decision findings to the parent; Merriam acts on decisions

A decision the run's parent has to make, or has made, now reaches the agent that acts on it.
Two paths were broken.

A Burler `decision`-class finding went to a fixer that could neither ask the parent for the decision nor record it, so it was re-raised and re-deferred every round.
A decision recorded with `lyx loom decision add` reached Merriam as a `[creel]` line from `ly:LOOM` that its stencil refused: the sender was not its parent, and the text told it to read `decision-record.md`, which the Master stencil does not allow.

Closes #853
Closes #849

## Burler fix row

- When the fix row is told a parent notifier, it reads the round's accepted review through `burlerengine.ParseCurrentReview`.
  If the review holds `decision`-class findings, it posts one notice to the parent.
  The notice names each finding's id, summary and fix, the review path, and the way forward, `lyx loom decision add`.
- The notice is posted once per fresh fix call, never when a live fixer is resumed.
  A failed post is logged at Warn and never changes the round's outcome, since the findings stay in the review on disk.
- A routing row excludes `decision`-class findings from the fixer's work.
  A review whose only findings are of that class is treated as a review with no findings: the gates run once, and if they pass the fixer is skipped.
  The report says the decision findings went to the parent.
- A spawned fixer's work line names the class as not its own: it neither fixes, disputes nor defers such a finding.
- A row told no notifier, such as `lyx burler run`, posts nothing and keeps every finding as fixer work.
- Loom's wiring hands the row a notifier that posts to the prime's orch as the `LOOM` program, as the run's stop notices do.
  The notifier reaches the row through `shedrecipe.Env`, not a row field.

## Decision notice and Merriam

- The notice `lyx loom decision add` sends each live agent opens with a fixed phrase, names the record and the entry's heading, and inlines the entry's decision and rationale.
  The committed record stays the authority for an agent that reads it.
- The parent directive's parent variant admits a `[creel]` line from the `LOOM` program that opens with that phrase, as a message carrying a decision of the run's parent or the operator.
  Every other message from a non-parent session stays untrusted.
  Creel accepts a program claim only from an orch pane or a process in no pane, so no task agent can sign as `LOOM`.
  The rule sits in the shared directive, so it covers every spawned role, not only Merriam.
- The Master stencil tells Merriam that the decision reaches code through the forks it spawns afterwards, which inherit it, and that it reads no record for it.
  While its own fork or fixer runs it ends its turn with no tool call.
  With no batch left to begin and no fixer running, it tells its parent by `SendMessage` that the decision reaches the code only through a gate fixer or a plan amendment.
- The webster implementer and verify-fix bodies tell a fork that a parent decision in its inherited context wins over its card where they differ.

## Design notes

- Merriam edits no source, so the decision can only act through forks.
  A fork already running when the notice arrives does not see it; the next fork does.
- A parent that never records the call sees one notice per round while the reviewer keeps raising the finding.
  This is bounded to one notice per round and is the intended pressure to decide.
- Out of scope: the Bouncer's stop rule, the review rubrics, creel's sender rules, the arbiter's notice path, and how the decision record is written or checked.

Package docs for `burlerengine`, `shedadapters`, `parentdirective` and `websterengine` are updated.

## To check by hand

- Run a live Webster run, record a decision with `lyx loom decision add`, and confirm Merriam accepts the notice and a later fork acts on it.
  Tests cover the notice text, the rendered directive and the stencils, not a live creel delivery.
- Run a Burler review that raises a `decision`-class finding and confirm the orch receives the notice.
