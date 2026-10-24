# Decision record — decision-reaches-agents

## Goal

A decision the run's parent needs to make, or has made, reaches the agent that acts on it.
A Burler `decision`-class finding goes to the parent instead of a fixer that can only defer it, and a decision recorded with `lyx loom decision add` reaches Merriam in a form it accepts and acts on.

## Scope

- In: the Burler fix row's handling of `decision`-class findings, in `internal/shedadapters`, `internal/burlerengine`, `internal/shedrecipe` and the loom wiring that tells the row its parent.
- In: the decision notice `lyx loom decision add` posts (`internal/loomcli/decision.go`), the parent directive stencil every spawned role carries, the webster Master stencil and the webster implementer and verify-fix bodies.
- In: the package docs of the changed packages.
- Out: the Bouncer's stop rule and the review rubrics; the creel sender rules; the arbiter's notice path; any change to how the decision record is written or checked.

## Decisions

### D1: The Burler fix row sends a round's decision-class findings to the run's parent

Decision: when the fix row is told a parent notifier, it reads the accepted review of the round through `burlerengine`'s parser and, when it holds `decision`-class findings, posts one notice to the parent naming each finding's id, summary and fix, the review path and the way forward, `lyx loom decision add`.
The notice is posted once per fresh fix call of a round, never on the resume of a live fixer.
A failed post is logged at Warn and never fails the round, since the findings stay in the review on disk.
Loom's wiring tells the row a notifier that posts to the prime's orch as the `LOOM` program, as the run's stop notices do.
Rationale: the round driver, not the fixer, can reach the parent, and the parent alone records the call; the new edge is one notice per round, bounded to the findings of an accepted review.

### D2: A routed decision-class finding is no fixer work

Decision: a fix call whose row routes decision findings excludes `decision`-class findings from the fixer's work.
A review whose only findings are of that class is handled as a review with no findings: the gates run once, and a passing gate skips the fixer with a report saying the decision findings went to the parent.
A spawned fixer's work line names the class as not its own: it neither fixes, disputes nor defers such a finding.
A row told no notifier, such as `lyx burler run`, keeps every finding as fixer work.
Rationale: a fixer that can neither ask for nor record the decision can only defer it again; the exclusion is bounded to rows that send the findings on.

### D3: The decision notice carries the decision's text

Decision: the notice `lyx loom decision add` sends each live agent of the run opens with a fixed phrase, names the record and the entry's heading, and inlines the entry's decision and rationale.
The committed record stays the authority for an agent that reads it.
Rationale: Merriam may not read the decision record, so the text must arrive in the notice.

### D4: Every spawned role accepts the decision notice as its parent's

Decision: the parent directive's parent variant admits a `[creel]` line from the `LOOM` program that opens with the decision notice's phrase, as a message carrying a decision of the run's parent or the operator.
The admission covers only that notice: every other message from a session other than the parent stays untrusted.
Rationale: creel accepts a program claim only from an orch pane or a process in no pane, so no task agent can sign as `LOOM`; one rule in the shared directive covers Merriam and every other spawned role.

### D5: Merriam hands a decision on through its forks

Decision: the Master stencil tells Merriam that a decision notice reaches the code through the forks it spawns afterwards, which inherit it, and that it reads no record for it.
While its own fork or fixer runs it ends its turn with no tool call; with no batch left to begin and no fixer running, it tells its parent by `SendMessage` that the decision reaches the code only through a gate fixer or a plan amendment.
The implementer and verify-fix bodies tell a fork that a parent decision in its inherited context wins over its card where they differ.
Rationale: Merriam edits no source, and a fork's prompt tells it to do only what its file says, so the file must name the decision.

## Constraints

- `PATTERN-sole-parsers`: the fix row reads the review only through `burlerengine.ParseCurrentReview`.
- `PATTERN-stencil-ownership`: stencil text stays in the stencil files, read at call time.
- `PATTERN-recipe-setting-lookup`: the notifier reaches the row through `shedrecipe.Env`, never a row `Env` field.
- `PATTERN-parent-directive`: the admission line lives in the parent directive stencil, rendered by `internal/parentdirective`.
- `PATTERN-test-economy`: new behavior extends existing tests where one covers the surface.

## Auto-mode assumptions

- The run's parent of a loom or darn run is the prime's orch, which `orchcli.NotifyPrime` reaches.
- A creel program post is typed in full into the receiver's pane, so the inlined text reaches the agent.

## Open risks

- A parent that never records the call sees one notice per round while the reviewer keeps raising the finding.
- A fork already running when the notice arrives does not see the decision; it reaches the next fork.

<!-- lyx:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/code/round-1-review.md; fixer report: _lyx/reviews/code/round-1-fixer-report.md.
  - live-fixer-resume-notice-untested (class scope, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Code-Review -->

## Acceptance criteria

- A fix call over a review with a `decision`-class finding posts one notice to the parent naming the finding's id, summary and fix, and a failed post leaves the round's outcome unchanged.
- A fix call with no notifier posts nothing and keeps the finding as fixer work.
- A routed review whose only findings are `decision`-class skips the fixer when the gates pass, and its report names the routing.
- A spawned fixer's work line, on a routed row, names the `decision` class as not its own.
- The decision notice line inlines the entry's decision and rationale and opens with the fixed phrase.
- The rendered parent directive names the decision notice's phrase and the `LOOM` sender.
