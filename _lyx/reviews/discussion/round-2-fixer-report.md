# Fixer report: discussion, round 2

## Changes

- F1, fixed differently.
  D10 now states that Finalize performs the parent-side merge and batten's Land row only pulls, deploys and prunes, so no join uses a Land `done` commit.
  The first source is an exact join: the run's landed task commit is the `Warp-SHA` trailer of the newest trailer-bearing commit in the tag's history, and the landing is the first-parent commit that is it, has it as a parent, or has its tree.
  The second source is a time join bounded by two Finalize-written records: the checkpoint commit, written before the merge, and the board commit marking the entry done, written right after it (`markTaskDone`, `internal/landingshed/finalize.go:281`).
  The exact join runs for every run before any time join, no commit is claimed twice, and two runs claiming one commit both become unmapped with a finding.
  The Rationale now says what each source guarantees.
  Proposed change not made: matching the landing's merged-in task-branch commit only against the pending-records commit's trailer.
  Finalize's merge may squash (`landingshed.Config.Squash`, `internal/landingshed/finalize.go:217`), and a squash commit has no merged-in parent, so the join also matches by tree.
  The pending-records commit is conditional (`commitPendingRecords`, `internal/fabricengine/remove.go:574`), but every fabric weft commit carries the trailer (`internal/fabriccli/weft_verbs.go:178`), so the join reads the newest trailer in the tag's history.
  Acceptance criteria now cover the no-op checkpoint mapping through the trailer, a squash landing, a foreign landing inside the window and a double claim.
  The support log's scenarios entry and Rejected alternatives were updated.
- F2, fixed as proposed: `hand` is a mapped run whose loom history holds no Finalize `done` entry.
  A run whose Finalize is done but whose batten Land never reached `done` keeps its label kind and carries a `land-incomplete` flag, absent when the batten source is absent.
  The acceptance criterion was updated, another was added, and the support log's goal-scope entry and Q7 were updated.
- F3, fixed as proposed: D5 states that each owning package gains a tolerant parse-from-bytes function separate from any strict one, and names `shedrun.ParseSeed` as the strict, live-called instance the report never calls (`internal/shedrun/seed.go:121`, `:131`).
  D9 points at that rule, and Rejected alternatives records loosening `ParseSeed`.
- F4, fixed as proposed: Notes say `alignment` owns the verdict format and the report reads it through the tolerant function D5 adds there, and `shedrun.AlignmentDir` is described as a place in the tag tree.
  Premise confirmed at `internal/alignment/records.go:173` and `:184`.
- F5, fixed as proposed: D4 defines the run's own fresh seed as the newest for the slug after the previous archive tag's commit and at or before this tag's, absent when none falls in that interval.
  Added the acceptance criterion.
- F6, fixed through the review's fallback.
  D7's presence test now accepts the friction dir or any archive sibling.
  No committed record shows Tier 2 was on: Tier 2 is `loom.yaml`'s `friction` key (`internal/loomcli/wiring.go:523`), which no tag tree holds, and the Friction-Reflect row only logs its status (`internal/loomshed/frictionreflect.go`).
  So the bias is stated in D7 and Open risks.
  Added the archive-sibling acceptance criterion.
  The proposed Tier-2-record criterion was not added, since that record does not exist.
- F7, fixed as proposed: the all-sources criterion now says it checks counts only.
  Added or extended criteria for D10's label mapping with its timing and pruned-entry case, D9's `accepted` outcome and parent-review outcome, D6's `pr-gate` wake and the always-absent Land ready-tasks kind, D7's leading-clause truncation and quoted-string placeholder, and D11's exactly-half case being `aged`.

## Disputed

None.

## Deferred

None.
