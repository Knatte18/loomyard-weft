---
round: 3
focus:
  - "Card 10 (`_lym/plan/10-teardown-pushes-first.md`) adds `Teardown.Push` and `Teardown.Level` to seam lists. Settle whether each seam appears in exactly one of `internal/testkit/envkit#nilLegal` and `internal/loomcli#intentionallyNil`, given that `envkit.NilSeams` skips every `nilLegal` path and loomcli's `TestWire_EverySeamFilled` fails for an `intentionallyNil` key that `NilSeams` never reports. Check battencli's wire and `FullEnv` still pass."
  - "Card 1 (`_lym/plan/01-fabric-owed-unrecorded.md`) has `checkOrigin` delete the local tracking ref of a branch gone from origin whose entry is ended. Settle whether the plan gives an implementer a way to delete only that ref: `gitrepo.Repo.DropRemoteTracking` also unsets `branch.<branch>.remote` and `.merge`, and PATTERN-gitrepo-client-boundary puts remote-tracking ref writes in `gitrepo` under the remote-refs lock. Check that any new function is targeted by a card, listed with `internal/gitrepo/doc.go` as the doc-go-unlisted check requires, and passes the plan gate."
  - "Settle whether the branch keeps its upstream after that delete, so card 2's `RunCommitsAhead` and `TipsOnOrigin` read it as level and held rather than ahead and not held, and card 10's push-then-Level loop and the dobby restart end. Check also that the next push no longer takes `checkOrigin`'s `ErrNoUpstream` path that recreates a branch origin deleted."
  - "For any test the plan adds or extends to reach the tracking-ref delete, check that a named input reaches it through the public surface and that no separate per-symbol test duplicates a covering test."
---

Entries cover only the two cards the latest review touched, so the settled remainder of the plan is not re-reviewed.
