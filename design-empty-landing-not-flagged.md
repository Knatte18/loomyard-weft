# A docs darn can land with an empty diff and its report only in the change description

`empty-landing-not-flagged` · Task · bug, landing, darn · starting · Seed-IDE

Depends on: [`single-repo-fabric-mode`](design-single-repo-fabric-mode.md)

## What was seen (2026-11-18)

`comment-trim-second-pass` (darn, profile `docs`) measured comment shares and wrote the result only into its change description (`_lym/landing/summary.md`).
It committed nothing to the repository, so Land landed an empty diff and dobby reported "landed as 64e58795", which was main's tip.
The finding then lived only in the archived records tag (`archive/comment-trim-second-pass/...`), where no one looks.

## Idea

- The darn writer stencil for profile `docs` requires the report as a committed file under `docs/research/`.
- Land refuses, or at least flags in dobby's line, a run whose code diff against its parent is empty.
