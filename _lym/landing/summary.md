# Landing notice: a beam member's land_notice off means no line

A beam member used to get its landed line whenever the beam's pull request was recorded, whatever its `land_notice` read.
That exception is gone.
`land_notice` off now means no line for every run, beam members included, so per-task beam settings follow one rule.

## What changes

- `rememberRuns` in `internal/dobbyengine/landnotice.go` decides a line only from the run's `land_notice` reading.
  While the beam's pull request is recorded, a line that is sent still ends with the beam's follow-up `lym beam describe <slug>`.
- `LandNoticeState.Member` is removed, with the board read that fed it: `isBeamMember`, `landNoticeTasks` and the `landNoticeBoard` failure episode.
  Nothing else read them.
  A state record an older binary wrote with a `member` key still decodes; the key is ignored.
- The package doc in `doc.go`, the `Deps.PullRequestRecorded` comment in `loop.go` and the doc comments in `landnotice.go` drop the exception.
- `contracts/orchrole/role-beams.md` tells the orch that a member gets the notice only when its `beam.land_notice` reads on.
- `TestLandNotices` drops the rows that pinned the exception and the board-membership plumbing.
  It gains rows pinning that an off member gets no line under a recorded pull request and an on member gets the follow-up.

Keeping the pull request's description current without the orch is out of scope: note `beam-pr-describes-itself`.

## Design decisions

- No exception, by the operator's decision: `land_notice` is the only switch for the landing notice.
  A beam whose orch relied on every member's notice to refresh the pull request description must turn `land_notice` on for its members.

## Not in this diff

The board task also lists findings `per-task-settings-one-lookup` landed open:
`reinterpret-discovers-worktrees-and-anchors`, `per-task-keys-declared-at-several-sites`, `registration-test-lacks-keep-marker`,
`duplicate-row-in-prepare-run-settings`, `carry-misses-runs-in-flight` and `beam-project-notify-on-land-dropped`.
This diff touches none of their sites, so they remain open.

## To check by hand

- Finding `merge-danger-two-way-miscalled`, left open: the door and blast radius below are called from this diff, and a reviewer should confirm them.
- Nothing was run by hand beyond the tests; no beam with a recorded pull request was exercised live.

## Merge danger

**Door:** two-way
**Blast radius:** dobbyengine
