---
round: 2
focus:
  - "internal/fabricengine/scratchlink.go, the sibling census behind `unseedScratchExclude` and `scratchLinkedInSibling`: settle whether the root `/.scratch` exclude line is kept for every sibling worktree whose root `.scratch` entry exists, whether link, real directory or file, and how a Lstat error other than not-exist is answered."
  - "internal/fabricengine/scratchlink.go, `unlinkScratchLink` and the wire path at a subpath anchor: settle whether a root `.scratch` that is not a link into the pair's weft worktree is left as found with its exclude line kept and a way forward reported, and whether the legacy root link is unlinked only through the destructive gate."
  - "internal/fabricengine/anchorchange.go, `unwireAtSource` ending in `unseedScratchExclude`: settle whether the old anchor's `.scratch` exclude line is dropped and the new anchor's line present after an anchor change, and whether a test pins both."
  - "internal/cleanupcli/segments.go, the scratch segment joining `location.AnchorRel` onto the worktree path: settle whether a subpath-anchor hub is read at the prime's anchor and whether a test fails if the path reverts to the worktree root."
  - "internal/fabricengine/doc.go and contracts/specs/loom-plan-spec.md: settle whether the `lym fabric remove` paragraph and the plan spec's transient draft location match the code's anchor-based `.scratch` placement."
---

Round 2 re-reviews the committed diff after the round 1 fixes.
