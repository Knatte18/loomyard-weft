```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: I checked whether the fixed text still agrees with the sibling decisions, criteria and cross-references it touches.

**Twins left unchanged:** the D3 Bound (line 57) and the D8 acceptance bullet (line 190).

**LOW 1: the D3 Bound still describes the old root-entry rule**
- Changed text: D3 line 51 now tests only tokens whose first segment is "an entry git tracks at the worktree root".
- Problem: D3's Bound at line 57 still says what goes untested is "a token under a root entry the worktree lacks (a Bolt path, an import path)". Under the new rule, a token under an untracked root entry that does exist on disk is also untested. Examples are `.scratch/…` and `.dev-bin/…`, and `.scratch` is present at this worktree's root. The Bound no longer names everything the rule lets through.
- Evidence: line 51 "an entry git tracks at the worktree root" against line 57 "a token under a root entry the worktree lacks".

**LOW 2: the D8 acceptance bullet doesn't cover the new test-code rule**
- Changed text: D8 lines 129–130 now classify a member glyph declared only in `_test.go` files as test code.
- Problem: the D8 acceptance bullet at line 190 still only says "a test-only card and a new package pass". No scenario covers the case behind F3: a test-only card that edits a test function by member glyph in a package that also holds production code, such as `internal/loomshed#TestNewDiscussionGate`.
- Evidence: line 129 "a member glyph that quarry reports declared only in `_test.go` files" against line 190 "a test-only card and a new package pass (D8)".

The other changes agree with their surroundings:
- The D1 ways forward and the D2 darn order agree with the D1 and D2 acceptance bullets.
- The D4 resolution sentence agrees with the Constraints bullet on `internal/planglyph` and with lines 76–77.
- The D5 trigger wording agrees with D5's Bound and its extended acceptance bullet.
- The D7 tier keys agree with D7's Bound.
- D9 now carries the index that D8's new classification needs, and D9's two builders agree with the review's cited files.
- The D10 chair wording matches the chair stencil, which includes `loom-template-discussion` whole, and the D10 acceptance bullet was updated to match. The Constraints bullet on D10 still agrees.
- The new D6 Auto-mode bullet agrees with D6.

Tool calls:
- Read the changed `_lym/discussion/decision-record.md` once. The fixer edited it with a perl script, so my context didn't hold the changed text.
- Listed `contracts/stencils/loom/` and read the header of `loom-template-discussion.md`. This confirmed it is the Discussion writer stencil the chair includes, which D10's new wording relies on and my context lacked.
