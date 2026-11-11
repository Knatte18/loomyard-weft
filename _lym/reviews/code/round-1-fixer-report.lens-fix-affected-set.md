```yaml
lens: fix-affected-set
outcome: reported
```

No findings on the two set-premise findings, F1 and F6, except the LOW entries below. I did not check F2–F5, whose premises aren't sets.

**F1: sites that write `StartedAt`**
- `rerouteStuck` writes it on three entries: the missing-`OnReroute` fault, the reroute and the reroute-capped halt (`reroute.go:49,59,70`).
- The fix pins the reroute entry (`reroute_test.go`, `assertCallTimed(t, last)`) and the capped entry (`TestRun_RerouteCapHaltsOnFourthReroute…`).
- The done and stuck-bounce entries are pinned in `step_test.go`.
- The stuck-halt entry is pinned in `TestStep_StuckWithNoOnStuck`.
- The fold keeps the first call's `StartedAt` (`historyfold_test.go` `st1`).
- **LOW:** the missing-`OnReroute` fault entry at `reroute.go:49` carries `StartedAt` and has no assertion.
  - `TestStep_RerouteWithoutOnRerouteFailsRun` fails the run and does not read the history.
  - The review's own search names "stuck, reroute and reroute-capped", so this is outside what the review asked for.
  - The entry feeds the same idle-gap computation, since `appendOrFold` writes it into the persisted history.

**F6: docs that list the api-error paths**
- **LOW:** `internal/sheddriver/doc.go:150` says "The error past them, or a model the provider rejects, is a hand-back with cause `api-error`."
- **LOW:** `internal/shedarbiter/doc.go:26` says "ends on an API error, its re-sends spent or its model rejected".
- Both list the same paths as `engine.go` and `overview.md`, omitting the capped skill-load hold, so they are the same kind of stale sibling.
- Both are at the shed layer. The capped hold surfaces there as a start error ending the run `api-error`, so whether they must change is a judgement call.
- **Checked, no gap:**
  - `shuttleengine/doc.go:516-517` already covers the cap.
  - `wait.go:20` and `apierrorretry.go` describe the in-run re-send path, not a start, so the cap does not belong in them.
  - `spec.go:78-80` and `wait.go:249` do the same.

**Calls made**
1. Read `internal/shedengine/reroute.go` 38-76. My context lacked its current content, to enumerate the `StartedAt` sites.
2. grep for `api-error`/`OutcomeAPIError` across non-test `.go` and `.md` files. This was F6's own search, to find every doc that lists the paths.
3. Read `internal/shuttleengine/doc.go` 512-520, to confirm the cap sentence is present.
4. Read `internal/sheddriver/doc.go` 146-152, to see the wording of the sibling mention.
