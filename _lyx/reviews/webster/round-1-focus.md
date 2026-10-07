---
round: 1
focus:
  - "Card 1: every weft-worktree birth path in `internal/fabricengine` creates `.weft/`, reconcile heals a missing one, and `ReadOriginFor` refuses a record without it; settle whether any birth path (add, checkout, reconcile recreate) bypasses the creation."
  - "Cards 2-4: the shared weft-branch resolver (local, then origin, then none) is the only resolution path in `checkout`, `reconcile` and `add`; settle whether any caller still forks from a local-only check."
  - "Cards 2-4: every branch creation, origin-tracked creation and fast-forward appends to the verb's `*Mutations` record, rollback of an origin-tracked branch goes through `deleteBranch`, and nothing deletes or force-pushes an origin branch; settle this for each new mutation site."
  - "Cards 2-4: a weft repo with no `origin` skips origin steps, while an unreachable or failing origin returns an error naming `origin`; settle that no path silently forks instead."
  - "Card 4: the live-pair rule's cases (adopt origin weft and warp, fast-forward a behind local weft, keep an existing origin record) match the plan's card text; settle behavior for a diverged local weft and a local weft ahead of origin."
  - "Cards 5-6: the config-changes diff is read-only and never carries the changes; Finalize queues exactly one orch notice at Done, and `landingshed` names neither fabric side and derives no path; settle that wiring runs through `loomcli` closures only."
  - "Git access: no new `//gitexec:raw` site, and all warp/weft git goes through `gitexec.Run` inside `internal/fabricengine`."
  - "Per-card mechanical check: for each card's groups, confirm the type-specific check ran against that group's own targets."
  - "Test economy: the integration tests in `internal/fabriccli/adoptremote_integration_test.go` and the extended scenario tests; settle whether any new test duplicates an existing test's coverage, naming the covering test, and whether any added behavior lacks coverage."
  - "Comment conventions: new or changed doc comments follow the repo's `CLAUDE.md`, `PATTERN.md` and surrounding code; docs (`doc.go`, `docs/overview.md`, `PATTERN.md`) land in the same commit as behavior changes."
---

Seed focus for round 1, derived from the plan's Shared Decisions and card index.
The review range is `git diff $(git merge-base main HEAD)..HEAD`, since no rework round exists.
