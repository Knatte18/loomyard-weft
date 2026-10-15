# Discussion hands Plan a list of the relevant code

Plan-Write was the second-largest token cost after the Webster forks, because the plan lead and its forks rediscover code the Discussion writer already read.
The loom Discussion-Write lead now writes that reading down as `_lyx/discussion/relevant.md`, beside the decision record and the support log.
Plan-Write reads the decision record, then the list, then the code the list names, and explores further only where it needs to.

## What changes

- `relevant.md` is a third Discussion-Write output.
  It is a bullet list of backticked worktree-relative paths, optionally with backticked member names, each followed by ` — ` and one line of why.
  It carries no glyphs, signatures, `file:line` citations or excerpts, and records only what the writer already read.
  `contracts/specs/loom-discussion-spec.md` pins the format and states that a darn writer does not write it.
- `loomengine.DiscussionRelevant` declares the path.
  The lead seat lists it in `Outputs`, so the step's completion waits for it.
- `discussionparser.CheckRelevant` checks a present list: at least one entry, no stray lines or nested bullets, every entry a backticked in-worktree path plus a non-empty reason.
  A missing file is not a finding and a duplicate path is tolerated.
  It does not check that a named path or member exists.
- The loom discussion gate and the loom branch of `lyx loom validate-discussion` both call it through `ValidateLoomDiscussion`, in line with `PATTERN-gate-self-check-parity`.
  `Validate` is unchanged, so darn validation and `lyx loom decision add` never see the file.
- The Discussion stencil asks for the file in Step 5 and "What you may write".
  The Plan stencil's Step 1 reads it first when it exists, and a missing list is not an error.
- A relaunched writer still finishes all three files: `twilldirective.Values` gains one host-supplied value, rendered by `twill-relaunch.md` in place of "the reviewed files".
  `ReviewedFiles` is unchanged, so the lens forks still review two files.
- The spec's "sole input" wording becomes: the decision record is the design input, `relevant.md` the code head start.
  `## Notes for the plan writer` keeps design hints only.
  `internal/loomengine/plan.go` and `internal/discussionparser/doc.go` are updated to match, with `docs/overview.md` and the test budget.

## Design decisions

- No new role between Discussion and Plan, and no extra Discussion search for Plan's sake.
- No reviewer reviews the list: Discussion-Review, the parent review and the lens forks are untouched.
  A list made stale by a review fix costs Plan at most one extra read.
- Absence is tolerated by every check, so runs whose Discussion finished before this lands keep passing later gates.
  Existence is enforced only by the lead seat's `Outputs`.
- The path rule rejects entries starting with `/`, a backslash, `~` or a drive letter, and entries with a `..` segment, whatever the host OS.
- The sibling task `engine-model-choice` rewrites `DiscussionTable`'s model resolution; this change only adds one entry to the lead seat's `Outputs` there.

## To check by hand

- The Plan host sets `WriterFork` and renders no relaunch part, so it supplies no value for the new relaunch marker; check that wording in `twill-relaunch.md` and `internal/loomengine/twill.go` reads right for both hosts.
- Check that the `RelevantPath` wire assignment in `internal/loomcli/wiring.go` is covered by a test; a plan review flagged it as untested.
- Check the test for the discussion gate's `RelevantPath` guard in `internal/shedrecipe`, added late after a review flagged the guard as untested.
- Stencil and spec wording (Discussion Step 5, the relaunch wording, Plan Step 1, the spec's format section) is covered by review, not by tests.
- The saving is not measured.
  Compare the plan lead peak context and the Plan cost with `tools/tokencount/tokencount.sh` over a few runs before and after.
- The final `lyx gate test --tags integration ./...` was not rerun after the last fix; run it before pushing.
