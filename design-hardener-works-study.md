# Study whether the hardener finds and fixes real faults

`hardener-works-study` · Note · research, burler, fabric

## Goal

Find out whether the hardener recipe does what it is for: drive the modules its focus names, find real faults, fix them with tests that hold, and converge.

## Evidence

- The first fabric campaign, `harden-fabricengine`, landed as `37649c772` after three rounds.
- The second, `harden-fabric-single-repo-verbs`, the one-repository fabric campaign with the fabric verbs in its sandbox, should have landed before this runs, so both campaigns are judged.
- Each campaign's round files are under its records worktree at `_lym/reviews/harden/`: `round-<n>-focus.md`, `round-<n>-review.md`, `round-<n>-acceptance.md`, `round-<n>-usage.yaml` and the snapshots, beside its decision record and the landed commits.

## Questions

- **Findings**: for each finding, is it a real fault, a false positive or a matter of taste? Do its repro steps show it?
- **Fixes**: does each fix cure its finding? Does its regression test fail with the fix broken, as the fix rules demand? Was the package doc updated?
- **Focus**: did each round stay inside the campaign's focus? Did the focus files steer later rounds to what was left unverified?
- **Convergence**: did the Bouncer end the campaign on a sound verdict? Were deferred findings filed as open risks and on the board?
- **Misses**: which faults in the same code turned up after the campaign landed, outside it (such as the killed-seed way forward in `0e4782373` and the fetch prune in `f660ac101`), and could the campaign have reached them?
- **Cost**: tokens and wall time per round, from the usage files and the run's records.
- **Seen in the second campaign**: round 2's reviewer recorded smaller faults (O1 to O6) only as prose, since an incremental round admits only BLOCKING findings on changed hunks.
  Round 3's reviewer (sonnet, medium, 4 min) approved with half its focus undriven: the killed discard, the keep-local tag push, a failed step with no kill, and `adopt --archives` over real owner tags.
  A settled harden segment cannot be reopened (`lym loom goto` refuses), so the orch had no way to order another round. Judge each of these, and whether the remaining focus should block approval.

## Bounds

- Mostly a study of records and commits.
- At most three findings are reproduced, on a sandbox hub against the commit before their fix, one try each; no other trial runs.
- The report ends with a verdict and a short list of concrete changes to the hardener recipe or its stencils, each a candidate note.
