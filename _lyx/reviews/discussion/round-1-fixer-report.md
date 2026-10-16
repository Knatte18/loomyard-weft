# Fixer report — discussion, round 1

## Changes

- **F1** (fixed as proposed): D5's Bound now says an entry calling a tier compiled rather than run lets the plan's `## verify:` section, which webster's plan-level verify gate runs as written, leave that tier unrun and lets Plan-Review accept it, bounded by the Guard classification on a board-only file and by the card and batch gates, which still run the untagged and `-tags integration` tests.
- **F2** (fixed as proposed, first option): D1 states that every non-overturning finding, scope-adding ones included, licenses its card's scope as today; the rubric bullet now reads "licenses a card's scope unless it overturns a Decision".
- **F3** (fixed as proposed, whole-rework stop): on a decline or with no parent, the writer plans no further finding, writes no `coverage.md` and ends its session reporting the finding, so the seat's missing output (`ReworkTable`'s `Outputs`) halts the run for the parent. The Bound and the Acceptance bullet match. The support log's Rejected alternatives records why dropping only that finding was not chosen.
- **F4** (fixed as proposed, kept-guard-sentence option): D6's implementer bullet keeps the guard sentence that a module-wide, `tmux`-tier or `llm`-tier run goes through `lyx gate test`. The empty-list fallback now limits raw runs to the listed packages and to what that sentence allows. D7's rationale no longer says the neutral rule "stays correct". Open risks adds the implementer gap: with no list, a `tmux` or `llm` test that no gate step names may be run through `lyx gate test` instead of compiled.
- **F5** (fixed as proposed): D6 keeps the sentence that a `lyx gate test` step runs as written, whatever its `--tags`. The list's compile-only rule governs raw runs and runs that no gate step names.
- **F6** (fixed as proposed, `none` option): D6 states that the marker is required at every site and that an empty list renders `none`. The Constraints line on `PATTERN-stencil-ownership` and the empty-list Acceptance bullet now cover every render. The support log's Rejected alternatives records the optional-marker form.
- **F7** (fixed as proposed): the rework Acceptance bullet now pins the no-parent stop, the skipped `coverage.md`, and the rendered rubric's reviewer's-claim wording and its flag on an unrecorded Decision-overturning finding.
- **F8** (fixed as proposed): D5's Rationale keeps only the language-facts sentence. The prose-over-schema reason moved into the schema entry under Rejected alternatives, and the .NET mapping moved into Turn 5's `dotnet-verify-protocol` entry. The support log's Review rounds also records this round.

## Disputed

None.

## Deferred

None.
