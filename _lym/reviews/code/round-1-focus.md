---
round: 1
focus:
  - "internal/cleanupcli/scratch.go and segments.go: the diff changes only comments there, yet the change description says the cleanup segment now reads `.scratch` at the prime's anchor. Settle whether the segment's code already resolves the anchored path or still reads the root, and whether the claim holds."
  - "internal/fabricengine/scratchlink.go `unseedScratchExclude` and `scratchLinkedInSibling`: settle whether the sibling census can leave the root `/.scratch` line in place permanently, against the goal that an anchored hub has no root `.scratch` wired or excluded and that Reconcile moves the prime over. Check the census when the prime itself is the worktree being retired, when `List` falls back to `l.WorktreePath()`, and when the exclude content has a trailing newline (`strings.Split`/`Join` round trip)."
  - "internal/fabricengine/anchorchange.go: `unwireAtSource` now drops the old anchor's exclude line, but check the rollback path and the wiring at the new anchor. Settle whether a failed or rolled-back anchor change leaves the scratch link and its exclude line at the right anchor, and whether the new anchor's scratch step runs after the move."
  - "internal/fabricengine/reconcile.go, remove.go, unwire.go, junction.go: `withoutScratchLink` now drops the name `.scratch` from every on-disk junction scan at any anchor. Settle whether that skips a user's own `.scratch` junction or entry in an unexpected place, whether Unwire at a root anchor still keeps link and exclude line as before, and whether `Remove` still unlinks both the anchor link and a legacy root link before the directory removal."
  - "internal/fabricengine/scratchlink.go `retireRootScratchLink` and `wireScratchLink`: settle the order and failure handling when the root retire returns a leave or failure, whether the anchor wiring still runs, and whether the combined outcome via `and` reports both. Check that a root `.scratch` that is a real directory or another pair's link is never removed."
  - "internal/fabricengine/scratchlink_integration_test.go and the other changed tests: for each new or changed test, name any existing test that already covers the same behavior. Check that the retire-root, leave-as-found root entry, sibling-census keep, anchor-change exclude drop and legacy-root `Remove` behaviors each have coverage."
  - "docs/overview.md and the fabricengine `doc.go`: settle whether the docs the repository's constraints name for a fabric wiring change land in this diff, and whether their statements about the scratch link's location, exclude line and teardown match the code."
---

Seed focus for round 1.
The goal is the decision record's `## Goal`: no plan exists, so the amended board entry is the measuring stick.
The change description is a claim, and the cleanup segment claim is the most checkable one against the diff.
