---
round: 2
focus:
  - "In `_lyx/discussion/decision-record.md` D7, settle whether the way forward for a decision against an already-done card needs a new Card Index line in `00-overview.md`; `planparser.ParsePlan` builds cards from that index and `Rebaseline` (`internal/websterengine/rebaseline.go`) refuses any change to the overview file. Check that D7, Scope 'Out' and the acceptance lines agree on one landing path with a stated bound."
  - "In D1 and Open risks, check whether `report-head` running under a held run lock states a mechanical bound on a live fork or recovery strand writing, and whether the remote task branch moved by `ResetPairCode` is part of the stated bound."
  - "In D6, check when the `card_amended` mark clears relative to the recovery strand's own terminal record, what a failed recovery does to it, and which signal Master's ladder reads to take the `card_amended` branch after a recovery that itself failed."
  - "In D12, check that the migration hook states the outcome for a profile carrying both `weights.master_base` and `weights.orientation`, and that `startup_context` (a `weights` key) and `alone_above` (a profile-level key) are placed at their real levels in `internal/batcher/config.go` and `estimate.go`."
  - "Check D3 and D4 for the same guarantee about the tree a reset to start leaves, and D3's archive-only path for which `PlanReset` guards still apply, including removal of live recovery strands as `internal/webstercli/reset.go` does today."
---

The entries name questions to settle; they cap no severity and state no verdict.
