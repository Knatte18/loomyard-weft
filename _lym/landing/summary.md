# Stage the Code-Review reviewer's diff in pieces that each read whole

The Code-Review reviewer had to derive the task's diff itself.
On a large branch it wrote the whole diff to a scratch file, read it in spans, and had its last read refused for size.
That put about 3,600 lines of diff in its context.
Burler now writes the round's diff out before the reviewer starts, one piece per package, each small enough to read whole, and the explore step names every piece.

## What changes

- **`internal/burlerengine`**: `Profile` gains `Diff`, the unified diff the round reviews.
  `Engine.Review` writes a non-empty `Diff` into a `diff/` directory beside the reviewer's instruction files (`diffpieces.go`).
  Each package's files go in one piece, and no piece passes `diffPieceMaxBytes` (60,000 bytes, well under the roughly 25,000-token limit of an agent's file read).
  A package too large for one piece splits at file boundaries into numbered parts.
  A single file that is too large splits at its hunk starts, then at line ends.
  The explore step's new optional `diff_pieces` marker renders the new `burler-diff-pieces` stencil, which lists the pieces.
  It tells the reviewer to read them in place of the range's `git diff`, and to run any other command the rubric names, such as `git show` of a merge commit, as before.
  A round with no diff renders the same explore step as before. An assistant reviewer's explore step names the same pieces.
- **`internal/shedadapters`**: `BurlerHalfDeps.Diff` is a reader the review producer calls once per spawning call, setting the spawned profile's `Diff`.
  A failed read is logged, and the reviewer spawns with no pieces, which leaves it deriving the range from the rubric as before.
  A probed or resumed reviewer is not affected.
- **`internal/shedrecipe`**: the `BurlerReview` row takes the optional key `stage_diff`.
  When it is set true, the row hands the full patch of `Env.CodeSubject` to the producer, the same diff against the task's merge base that the Code-Bouncer reads.
  A subject that is not found, as in a standalone run, gives no diff.
  A row that sets the key while `Env.CodeSubject` is nil is refused when it is built.
- **Recipes**: the `Code-Burler-Review` rows of the tapestry, darn and sampler recipes set `stage_diff: true`.

## Design decisions

- Burler stays free of git: the review producer reads the diff through the existing code-subject seam and hands burler only the text, so standalone burler runs need no repository.
- The diff is read per spawning call, not when the row is built, because each round's fixer commits change it.
- The Code-Review rubric still defines the review range, so a merge-parent task's narrowed range and the `none` base still go through its own rules.
- The piece cap is a named constant, not a setting: it tracks a file-read limit, not an operator choice.
