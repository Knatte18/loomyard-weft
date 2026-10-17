# Bound landing PR lookups to the run; count only titled decision notes

A run's landing steps now act only on a pull request created at or after the run started, so a reused slug no longer adopts the earlier run's closed pull request.
The alignment agent's decision snapshot now moves only on a real decision for the task, not on a board note that merely names the slug.

Closes #692

## Pull-request lookup

- `landingshed.FindPullRequest` takes a lower bound and returns the newest pull request from head to base, in any state, whose `created_at` is not before it.
  An older pull request reads as none.
- The bound is the `started_at` stamp of the pair's own run seed, read in `internal/loomcli` through `shedrun.ReadSeed` and handed to `internal/landingshed` as a told value in `landingshed.Deps`.
- Every caller passes it: Publish, PR-Gate, PR-Align, Finalize's close, and the `findPR` seams of `lyx loom approve` and `lyx loom reject`.
  No landing step can find a different pull request than the one Publish created or refreshed.
- A pull request opened by hand during the run counts as the run's own, and a resumed Publish refreshes it instead of creating a second.
- A missing seed, or a seed with no parseable stamp, gives a zero bound that filters nothing and logs a warning.
  A seed that exists but cannot be read or decoded fails the wiring, and approve and reject refuse with the error.
- Publish stops Stuck when the bounded lookup finds none but the head branch has an open pull request created before the run started, since GitHub refuses a second open pull request for the same head and base.
  The reason names that pull request's URL and ends in a `way forward:` clause: close it on GitHub, then resume with `lyx loom start`.
  An older closed or merged pull request is ignored and Publish creates a new one.
  `FindPullRequest`'s return grows to carry that older open pull request from the same list response, so Publish makes no second GitHub call.

## Decision snapshot

- The snapshot hashes the decision record and the board notes whose title starts with `Decision for <slug>:`, case-sensitive, with the colon, so `Decision for foo:` does not match slug `foo-bar`.
  A note naming the slug elsewhere, or an entry of another kind, no longer moves it.
- `internal/alignment` declares the title form once, as a function of the slug.
  The orch role stencil renders it through a new template key filled by `internal/orchengine`, and the key is a required marker, so an operator override without it fails the render naming the marker.
- `decisionRecordedReason` is now one constant in `internal/alignment`, replacing the duplicate in `internal/landingshed/pralign.go` and `internal/loomcli/alignmentwiring.go`.
- `doc.go` of `internal/landingshed`, `internal/loomcli`, `internal/alignment` and `internal/orchengine` is updated to match.

## Design decisions to know

- A free-form decision note, including ones already on the board for a held task at landing, stops counting.
  An operator who wants the agent's approve held titles the note in the `Decision for <slug>:` form, or records the decision with `lyx loom decision add` when a decision record exists.
- The bound relies on the seed's stamp being this run's.
  A new pair for a reused slug gets a fresh seed only after `lyx batten cleanup --apply` removes the earlier run's `_lyx/shed/<slug>` directory, which batten's completed-slug refusal already requires.
  Batten's refusal itself is unchanged.
- The page-one assumption: GitHub's list endpoint, sorted by creation date descending, returns the run's own pull request within its first page.

## To check by hand

- Re-seeding a run after Publish has created its pull request moves the bound past that pull request, so the run's own open pull request reads as older.
  Publish then stops on it, and PR-Gate, PR-Align, Finalize's close and approve and reject find no pull request.
  The `internal/loomcli` module doc and the seed-restore guidance describe how to restore the seed's `started_at`.
  Check that the warning is clear enough, since PR-Gate's nil-lookup reason suggests reopening the pull request, which cannot help here.
- No run against live GitHub was made: the lookup bound, the older-open-pull-request stop and the first-page assumption are covered by fake-backed tests only.
