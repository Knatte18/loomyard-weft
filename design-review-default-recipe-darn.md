# Review and fix the unreviewed default_recipe change

`review-default-recipe-darn` · Task · enhancement, board, batten · running · Code-Bouncer

## Why

Task `default-recipe-darn` ran darn with profile `docs`: a Sonnet writer and no Code-Review.
It changed code in five packages (`battenshed`, `battencli`, `boardengine`, `boardcli`, `loomcli`) to add the hub setting `default_recipe` in `board.yaml` and resolve a task's recipe through `Board.RecipeOf`, so the change landed with no review.

## Goal

Review the commit `default-recipe-darn` landed on `main` as a Code-Review would, and fix what it finds.

- Find the commit by its subject, "A task with no recipe runs the board's default_recipe", and read its whole diff and the code around it.
- Judge it against the PATTERN entries, the repository's agent instructions and the code-quality and testing rules: one resolution point for an empty recipe with no literal left elsewhere, error handling where the board is opened, tests that pin the default and the override, and docs that state the setting once.
- Fix each finding on this task's branch; the Code-Review perch then reviews those fixes.
- A finding outside that commit's change becomes a board note, not a fix here.
- When the review finds nothing to fix, make no code change: record the review and its verdict in the change description and land.
