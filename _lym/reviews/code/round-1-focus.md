---
round: 1
focus:
  - "internal/lymcwd/lymcwd.go ResolveWithin and ownerOfRecordsSibling: does a cwd in the weft sibling always resolve to the owner's Location? Check a sibling whose owner has a recorded anchor, one with no marker, and a plain worktree whose name ends in the sibling suffix with the owner present."
  - "internal/lymcwd/anchor.go pathWithin and checkCwdWithinAnchor: do symlinks, the anchor's _lym link into the records sibling, a '..'-prefixed directory name and Windows case folding compare correctly? Does a cwd above or beside the anchor still fail?"
  - "ErrCwdOutsideAnchor's message changed: does any caller, test or doc still match or quote the old 'subtree' text, and does the new wording stay true for the strict gate in checkCwdAnchorGate?"
  - "internal/shuttlecli/cli.go: the resolver is chosen by cmd.Name() == awaitForksVerb. Does that reach only await-forks, and does the swap leave the rest of the pre-run (hub checks, error passthrough) unchanged for it?"
  - "Scope against the Goal in the decision record: it asks await-forks to accept a cwd anywhere under the anchor or resolve from the weft root, or else name the required directory. Does the diff stay within that, and does the new lymcwd surface follow the repo's cwd-resolution PATTERN and its docs (docs/overview.md, docs/shared-libs/lymcwd.md)?"
  - "internal/lymcwd/lymcwd_test.go TestResolveWithin: do its cases cover ResolveWithin's branches (no-marker anchor, non-sibling suffix name, outside git repo)? Does it duplicate an existing Resolve test? Is there coverage for await-forks resolving from a weft cwd?"
---

The Goal is the only populated section of the decision record, so the measuring stick is the board entry itself.
The focus entries point at the new resolver's edge cases, the changed sentinel text and the verb-specific dispatch.
