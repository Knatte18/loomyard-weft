---
round: 3
ledger:
  - key: guard-push-bypass-any-paths
    rounds: [1]
    status: resolved
  - key: archive-tag-plain-refs-tags-one-repo
    rounds: [1, 2]
    status: resolved
  - key: unpushed-commits-no-upstream-says-none
    rounds: [1]
    status: resolved
  - key: push-missing-fetch-refspec-no-way-forward
    rounds: [1, 2]
    status: resolved
  - key: pull-missing-fetch-refspec-reports-ok
    rounds: [1]
    status: resolved
  - key: killed-teardown-child-orphans-unnamed
    rounds: [1]
    status: resolved
  - key: guard-refusal-omits-sandbox-way-forward
    rounds: [1]
    status: resolved
  - key: pending-teardown-rerun-skips-owed-archive
    rounds: [2]
    status: resolved
  - key: archive-tags-share-one-origin-prefix
    rounds: [2]
    status: resolved
---

## Cross-round narrative

Round 1 found seven findings on the one-repository layout.
Round 2, under a different reviewer model, re-drove all seven fixes and found four; three were BLOCKING.
Round 3, under a third model, found nothing and re-drove the four open keys live.

- `archive-tag-plain-refs-tags-one-repo`: a discard with the code store's origin URL as `file://<path>` and as a symlink left only `refs/lym/code-tags/` and `refs/lym/tags/` refs.
  The relative spelling and the two-repository control were not driven.
- `push-missing-fetch-refspec-no-way-forward`: with the records store's fetch refspec unset, every verb answered with the restore commands, and typing them made each succeed.
- `archive-tags-share-one-origin-prefix`: code tags sit under `refs/lym/code-tags/` and records tags under `refs/lym/tags/`; a second clone's stores held only their own side's tags.
  `adopt --archives` over real owner tags was not staged.
- `pending-teardown-rerun-skips-owed-archive`: the records half re-ran correctly after a kill.
  The discard half was not driven, so the key is resolved on partial evidence and round 4 re-drives it.

The live gate was red: `harden-live` started no test binary.
The operator's delegate ruled the round cannot approve with focus entries undriven, so round 4 drives them.
A new BLOCKING or MEDIUM finding in round 4 on any of these keys reopens it.
