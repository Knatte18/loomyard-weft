# Fixer report: discussion, round 1

## Changes

- F1, fixed as proposed: D10 maps through the checkpoint commit first, stated as conditional, then through the slug's batten Land `done` status commit joined to the newest first-parent parent-branch commit at or before it, bounded by the slug's archive tag; a landing matching neither is `unmapped`.
  The Rationale now says the checkpoint is conditional.
  Added an acceptance criterion for a landing whose checkpoint was a no-op, and corrected the support log's scenarios entry.
  Premise confirmed in `parentRecordsCommitter` (`internal/loomcli/landingdeps.go:308`), which commits nothing when the parent's run-records directory holds no file.
- F2, fixed as proposed: D5 confines unknown-field tolerance to the report's parse functions and states the bound that no live reader's acceptance changes; the fourth Constraints bullet leaves live-reader tolerance to `run-robustness`.
  The support log's board-fit entry was updated to match.
  Premise confirmed: `KnownFields(true)` at `internal/alignment/records.go:184` and `internal/loomengine/arbiter.go:154`, `DisallowUnknownFields` at `internal/state/state.go:163`.
- F3, fixed as proposed: D9 parses every record from git-object bytes through a parse-from-bytes function in the owning package and calls no path-based or locking reader; D6's `battenshed` classifier and D7's halt-note reader point at that rule.
- F4, fixed as proposed: D4 bounds the batten record at the newest commit still holding `status.json`, from the run's own fresh seed up to the next fresh seed or the resolved branch tip, absent when none holds it; added an acceptance criterion for a slug cleaned by `batten cleanup`.
  Premise confirmed at `internal/battencli/cleanup.go:218`.
- F5, fixed as proposed: D6 states the Land ready-tasks notice leaves only scratch markers and an untracked notice-log line, so the kind is always absent; Q12 is closed, the notice log is in Rejected alternatives, and Turn 1 and the goal-scope entry name it.
  Premise confirmed in `internal/battenshed/rownotice.go` (`ClearRunMarkers`, `NoticeLog`) and `internal/battenshed/doc.go:44`.
- F6, fixed as proposed: D7 names the friction dir and its reflection-archive siblings as the source, reports a run with no friction dir as absent, limits `no-note` to runs with the source, and counts a leftover halt note as its own stop.
  The Open risks bullet on halts resumed between reads now points at that rule.
  Premise confirmed at `internal/loomcli/halt.go:62`.
- F7, fixed as proposed: D5 states the composite rule for wake totals, D9 makes a segment with no directory absent, and D11 reports a week with no fresh or aged fix as no data.
- F8, fixed as proposed: added acceptance criteria for the superseding approve wake, the Land failure wake, goto and reroute LLM steps, the unaccounted resume, `escalated` and `circling-decided`, `no-note`, and the `--since` filter.
  Also added criteria for the composite wake mean, the leftover halt note and the absent segment, which this round's own D5, D7 and D9 changes introduced.
- F9, fixed as proposed, taking the review's first option: D11 blames `unmapped` commits the same way and reports them as their own `unmapped` share, never mixed into the fix share, since no label separates a direct-push repair from other direct pushes.
  D10 names direct pushes as `unmapped`, D12's markdown section shows both shares, and an acceptance criterion pins the split.
- F10, fixed as proposed: D9 gives the outcome precedence `circling-decided`, then `escalated`, then `accepted`.
- F11, fixed as proposed: D4 states blame runs once per blamed landing in range, so an explicit `--since` widens the cost linearly, and accepts any `--fresh-days` of 1 or more.
- F12, fixed as proposed: Q7 states D10's `hand` and `unmapped` split.
- F13, fixed as proposed: deleted the next-tag clause from D4's Rationale and the `tools/` clause from D1's Rationale.

## Disputed

None.

## Deferred

None.
