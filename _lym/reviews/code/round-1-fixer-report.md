# Fixer report, round 1

Gate: `lym gate test --tier 1 ./internal/fabricengine ./internal/cleanupcli` passed, 0 failed, before and after the self-check follow-ups.

## Changes

- **F1** (fixed as proposed): `scratchLinkedInSibling` in `internal/fabricengine/scratchlink.go` now `Lstat`s `<sibling>/<anchorRel>/.scratch` and answers true for any entry and for any error other than `fs.ErrNotExist`; its doc comment and the `unseedScratchExclude` one say "entry".
  Row `RealRootDirectoryOfASiblingKeepsTheRootExcludeLine` added to `TestScratchLink_SubpathAnchor`.
  The `doc.go` census wording went into the F2 commit.
- **F2** (fixed as proposed): the `lym fabric remove` paragraph of `internal/fabricengine/doc.go` now names the anchor link, the legacy root link at a subpath anchor, a `link_removed` at each, the claim limit and the target's teardown.
  The two census sentences nearby say "entry" instead of "link".
- **F3** (fixed as proposed): `TestCleanupVerb_ScratchSegmentSweepsTheAnchorOfASubpathHub` in `internal/cleanupcli/cleanup_integration_test.go` builds a `backend` hub, ages a child under the anchor's `.scratch`, and asserts `--apply` removes it and keeps the link.
- **F4** (fixed as proposed, second option): subtest `A move drops the old anchor's scratch exclude line once no worktree still has an entry there` in `internal/fabricengine/anchorchange_integration_test.go`; the first subtest's exclude list also requires `/sub/.scratch` and the sibling-kept `/.scratch`.
- **F5** (fixed as proposed): subtest `LegacyRootLinkAtASubpathAnchorIsUnlinkedWithTheAnchorLink` in `TestRemove_UnlinksScratchLinkBeforeWorktreeRemoval`.
- **F6** (fixed as proposed, one row more): `RootEntryThatIsNotFabricsIsLeftAsFound` rows for a real directory, a regular file and a foreign link on `Shape{Anchor: "backend"}`.
- **F7** (fixed as proposed): `scratchShapeCases`, the adopt rows of `TestWireJunctions_ScratchLink` and `TestReconcile_ScratchLink` run at `.` and `backend`.
  Helpers `scratchExcludeLine` (new), `writeLegacyScratchExclude` and `trackScratchFile` now build the exclude line and the tracked path from the anchor.
- **F8** (fixed differently): the proposed change, "anchor's" in `contracts/specs/loom-plan-spec.md:128`, was not kept.
  It made the spec say Webster's `create-not-done` skips the anchor's `.scratch/`, while `internal/planglyph/donecheck.go:29,104` skips only the root-relative prefix `.scratch/` and `DoneChecks` receives no anchor.
  The line now reads "worktree root's", the wording `donecheck.go:62` and `planglyph/doc.go:137` use.
  The part of the premise left open is under Deferred.
- **Self-check LOWs** (fixed as proposed): `removeRootLink` and `replaceLinkWithRealDirectory` comments and failure messages no longer say "root"; the first cleanup test builds `scratchRoot` from the anchor.

Commits: F3, F4, F2, F8 and the F1/F5/F6/F7 group are separate commits, F1, F5, F6 and F7 sharing one because they edit the same two files, plus one commit each for the F8 rewording and the self-check LOWs.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, MEDIUM: spec names the anchor, the skip code roots at the worktree root (F8) — fixed differently, see F8.
- fix-consistency, LOW: first cleanup test builds `scratchRoot` without the anchor — fixed as proposed.
- fix-proposal, MEDIUM: same F8 contradiction, merged with the one above — fixed differently.
- fix-affected-set, MEDIUM: same F8 contradiction, merged — fixed differently.
- fix-affected-set, LOW: helper comments and messages still say "root" — fixed as proposed for the two helpers.

## Disputed

None.

## Deferred

- **F8, the anchor part of the premise.** In an anchored hub no `.scratch/` stands at the worktree root, so the plan skip rule matches nothing there, and a draft at `<anchor>/.scratch/x` gets a `create-not-done` check.
  Carrying the skip to the anchor needs an anchor input to `DoneChecks` (callers outside this review's surface), a decision on whether card paths are root-relative or anchor-relative, and a reversal of `planglyph/doc.go:138`, which pins "a `.scratch/` below the root keeps its check".
  That is an operator decision on a real tradeoff.
