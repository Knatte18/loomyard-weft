# Orch delegates board intake to a one-shot fork
The orch checked the GitHub inbox by running `lyx board intake list` and `intake import` itself.
Both printed a full issue body, so one survey issue entered the orch's context twice, about 14k tokens of a 71k session.
The orch needs only numbers and titles to know whether the inbox holds anything, so the check is now cheap and the intake itself moves to a one-shot fork.

## What changes

- `lyx board intake list` prints `{"issues":[{"number":…,"title":…}]}` by default.
  `--full` prints the previous view: number, title, body, labels, url, comments and created_at.
  Filtering is the same in both modes: recorded issues, closed issues and pull requests are left out, and an empty inbox prints `{"issues":[]}`.
- `lyx board intake import` prints `{"slug":…,"issue":…,"labels":[…],"dropped":[…]}` instead of the entry.
  A fold prints the same, with `slug` the entry folded into.
  The already-recorded no-op keeps its `recorded` output.
- When the board write succeeds but the GitHub comment or close fails, the error envelope carries those four fields plus the `close` payload to re-run, in place of `entry`.
- The orch role stencil (`contracts/stencils/orch/orch-template-role.md`) tells the orch to run `lyx board intake list` at every wake and read only numbers and titles.
  An empty list ends the check, and a non-empty list goes to one one-shot fork.
  The stencil states the fork's three outcomes (new note, fold, close as noise), its write bounds and its one-line-per-issue return.
- The `ly:board` skill's Intake section matches: `list` is compact, `--full` shows bodies, a new note gets its type and module labels through `import`'s `labels`, and a label `upsert` on a folded entry carries the entry's whole label list.
- `internal/boardcli/doc.go` states the new list and import output.

## Design decisions

- Compact is the default and the full view is the flag, so an orch running the bare command never pulls a body into its context.
- The orch does not record an open intake fork.
  The inbox is the state: a fork lost to a context cycle leaves its issues open, and a later wake's check forks again.
  A second fork beside the first is harmless, since the CLI settles each issue once: a second import prints `recorded`, a close of an imported issue is refused, and an import of a closed issue is refused.
- An error envelope from `list` is neither empty nor non-empty: the orch starts no fork and handles it as a lyx bug.
- A fork is used rather than a fresh agent because it inherits the board policy and the operator's settled decisions.
- When an import's GitHub close fails after the board write, the fork re-runs the `close` payload once, since a recorded issue no longer shows in `list`.
  A second failure becomes a `failed` line carrying the error and the payload.
- Out of scope: `intake close`, `boardengine`'s import, fold and labelling rules, and `selfreportengine`'s GitHub calls.

## To check by hand

- A running orch keeps its rendered role file and its loaded `ly:board` skill until its next context cycle, so after deploy it does intake the old way until then.
- The stencil and skill wording is not pinned by a test.
  Check that the stencil's fold rule and its close-as-noise rule do not overlap: a duplicate that adds anything to an entry is folded into it, or becomes a new note when the entry is run-held.
- The intake fork has not been run against a live inbox.
