# PR-Rework starts a new plan generation per round

A PR-Rework round now retires the live plan and starts a fresh generation, instead of appending cards to it.
After a round, `_lyx/plan/` holds only that round's cards and a new `00-overview.md`, so Webster's prompt and context stay flat however many rounds a task takes.
Refusal-audit circled for this reason: four rework rounds grew its plan to 64 cards, which overflowed Master's launch prompt and exhausted the bounce budget.

Two review-stencil fixes ride along: the discussion and plan rubrics ask what a new verb or downgraded guard can skip or let through, and a Burler focus directive no longer forces the reviewer to read prior-round files before its own review is saved.

## What changes

- PR-Rework (`internal/loomshed/prrework.go`) moves the live generation into `_lyx/loom/rework/round-<N>/prior-generation/` before the session starts.
  The moved set is every `_lyx/plan/` entry (`archive-*/` included), Webster's durable run record, and the Plan-Review and Webster-Review run directories.
  The session writes a whole new plan, and the archive moves and the new plan land in one commit.
- `00-overview.md` gains an optional `first_card` scalar (default `1`), so card numbers stay unique across generations; `planparser` checks numbering against it.
- The `rework-plan` gate (`lyx loom validate-plan --rework`) now validates the whole new plan, plus `first_card` equal to the told number.
  `planparser.CheckAppendOnly` and the new-cards-only scoping are deleted, as is `websterengine.RebaselinePlanFingerprint`.
- `websterengine` gains a primitive that moves the run record into a told directory, so the next `lyx webster run` starts fresh.
- `PR-Rework.on_done` is now `Plan-Bouncer`, so a rework generation walks the same main line as the original plan.
  Plan-Bouncer carries a `skip_seam: rework-exempt` key that approves without a judge for an exempt generation.
- Plan-Review's answer key for a rework generation includes the round's `findings.md`; Webster-Review reviews only the live generation's own commits; Describe is told every generation's run record, oldest first.
- Docs: `docs/overview.md`, `README.md`, the recipe header, `lyx loom reject` help, the plan and shed-recipe specs, and a new Plan Generation Invariant in `CONSTRAINTS.md`.

## Design decisions

- **Mechanical review routing.** A generation is `exempt` only when every card carries only the `Prosa` label and no `Prosa` target is a source file of the plan's `language:`; anything else is `required`.
  A label the rework agent picks cannot be trusted to flag its own risky card, and `prosa-symbol-target` admits `Prosa` on Go files.
- **Skip bound.** The skip seam skips only when the live generation's round record says `exempt` and the classification re-run over the live committed cards also says `exempt`.
  A stale or forged record can at most skip review of cards that are themselves exempt.
  Generation 0 and a `first_card` with no matching round record never skip.
- **Crash safety.** `record.json` is written last in the archive step and marks it complete; an interrupted archive resumes into the same round directory, and the archive never moves a file the session wrote.
- **Removed guard.** Dropping the append-only check removes the guard against silently rewriting built cards; the built cards are no longer in `_lyx/plan/` when the session runs.
- **Accepted gap.** An exempt generation can still change behaviour through a `Prosa` card on a non-source file that agents follow as rules; Webster-Review still reviews the resulting diff.

## To check by hand

- A real `lyx loom` rework round end to end: only the integration tests drive it.
  Confirm `lyx webster run` starts a fresh run afterwards and Plan-Bouncer routes as expected for an exempt and a required generation.
- A run parked at PR-Rework under the old append-only rule is not migrated; restart it.
- `gate-list` is landing in parallel and replaces the recipe's per-row `gate` keys with a `gates:` list; whichever lands second keeps this task's routing and skip-seam changes beside the `gates:` form.
- `gofmt -l` flags `internal/lyxcwd/docslink_test.go`, `internal/shell/posix.go` and `internal/websterengine/strand_test.go`; this change does not touch them.
