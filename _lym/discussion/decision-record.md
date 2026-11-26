## Goal

```markdown
## Goal

Find out whether the hardener recipe does what it is for: drive the modules its focus names, find real faults, fix them with tests that hold, and converge.

## Evidence

- The first fabric campaign, `harden-fabricengine`, landed as `37649c772` after three rounds.
- The second, `harden-fabric-single-repo-verbs`, the one-repository fabric campaign with the fabric verbs in its sandbox, has landed as `2722a66ef`, so both campaigns are judged.
- The second campaign ran with the model rotation the template then set (opus, fable, sonnet).
  Hardener rounds now default to `sonnet[high]` alone, and a rotation is set per run through `hardener.reviewer` and `hardener.fixer` (`a66d41345`), so judge the rotation's cost and effect against that change.
- The five faults the second campaign found but could not raise were the task `harden-fabric-undriven-cases`, which has landed as `540e8c9b5`.
  Judge that fix as the campaign's follow-through.
- The cases round 3 left undriven (a discard or remove killed mid-operation, a fabric step that fails with no kill, and `adopt --archives`) were then driven by hand on a one-repository hub in the task `single-repo-undriven-cases`, landed as `dce6b8617` with the report `docs/research/single-repo-undriven-cases.md`.
  It found one fault the campaign's rounds had not: the killed run's `remove` record names `lym fabric remove <slug> --remote`, which answers `pair not found` once the pair is gone.
  The task `remove-record-rerun-pair-gone` fixed it (`cd8090733`) for a record a `batten cancel --remove` wrote, which now names that cancel; the fix was not driven live.
  Judge it as a miss: could the campaign have reached it? The keep-local tag push was not among the cases it drove, and a real `adopt --archives` takeover needs a second machine, so the tags were pushed by hand.
- Each campaign's round files are under its records worktree at `_lym/reviews/harden/`: `round-<n>-focus.md`, `round-<n>-review.md`, `round-<n>-acceptance.md`, `round-<n>-usage.yaml` and the snapshots, beside its decision record and the landed commits.

## Questions

- **Findings**: for each finding, is it a real fault, a false positive or a matter of taste? Do its repro steps show it?
- **Fixes**: does each fix cure its finding? Does its regression test fail with the fix broken, as the fix rules demand? Was the package doc updated?
- **Focus**: did each round stay inside the campaign's focus? Did the focus files steer later rounds to what was left unverified?
- **Convergence**: did the Bouncer end the campaign on a sound verdict? Were deferred findings filed as open risks and on the board?
- **Misses**: which faults in the same code turned up after the campaign landed, outside it (such as the killed-seed way forward in `0e4782373`, the fetch prune in `f660ac101` and the remove record's way forward in `remove-record-rerun-pair-gone`), and could the campaign have reached them?
- **Cost**: tokens and wall time per round, from the usage files and the run's records.
- **Seen in the second campaign**: round 2's reviewer recorded smaller faults (O1 to O6) only as prose, since an incremental round admits only BLOCKING findings on changed hunks.
  Round 3's reviewer (sonnet, medium, 4 min) approved with half its focus undriven: the killed discard, the keep-local tag push, a failed step with no kill, and `adopt --archives` over real owner tags.
  A settled harden segment cannot be reopened (`lym loom goto` refuses), so the orch had no way to order another round. Judge each of these, and whether the remaining focus should block approval.
  Round 4 again raised what it found outside the incremental scope only as prose; those are the O1 to O5 that `harden-fabric-undriven-cases` fixed.
  A suspected cause is the stencils, not the model: `contracts/stencils/harden/harden-rubric-review.md` asks for OK, WARN or FAIL on every focus entry,
  while `contracts/stencils/burler/burler-review-format.md` lets an incremental round raise only BLOCKING findings on changed hunks, so such a round can only approve. Judge that conflict.
  Since `8cf7f4775`, a later hardener round cannot approve with its focus undriven:
  the reviewer raises a `decision`-class, BLOCKING, `origin: focus` finding for each focus entry it left undriven or marked WARN,
  `burlerengine.CheckReviewRound` exempts it from the changed-file rule, and the arbiter and then the parent rule on it.
  That answers whether the remaining focus should block approval going forward; judge the campaign's rounds as they ran, and whether the new finding is enough for the rubric conflict.

## Bounds

- Mostly a study of records and commits.
- At most three findings are reproduced, on a sandbox hub against the commit before their fix, one try each; no other trial runs.
- The report ends with a verdict and a short list of concrete changes to the hardener recipe or its stencils, each a candidate note.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-26): Reproductions dropped

Decision: The study runs no reproductions; the report judges the three findings (c1 F3, c2 F2, c2 G3) from records and commits, and says that no reproduction ran and why.

Rationale: The entry's bound is at most three, and a task session cannot make the sandbox hub or old-build worktree they need; the orch runs no trials itself.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
