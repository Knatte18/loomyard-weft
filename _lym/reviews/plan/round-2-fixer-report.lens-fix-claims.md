```yaml
lens: fix-claims
outcome: reported
```

I checked the claims the fix added or rewrote against the code at HEAD.

**MEDIUM: Card 6, configcli bullet.** The new text says `TestConfigOverRealHub` "plants `plan_reviewer_fan` and `plan_fan` in the board `tapestry.yaml` and expects `plan_reviewer_fan` there after reconcile". The card doesn't make that outcome happen.
- **Evidence:** the `plan_fan` → `plan_reviewer_fan` rename is declared in loom's rename table (`internal/loomengine/config.go:1034`, `{From: "plan_fan", To: "plan_reviewer_fan"}`).
- Card 6 gives the tapestry module no rename. A `plan_fan` planted in `tapestry.yaml` has no migration to rename or drop it. The test's "gone" check for `plan_fan:` (configcli_integration_test.go:464) then fails, and so may tapestry's strict load.
- A hub that holds `plan_fan` holds it in `loom.yaml`. The test should keep planting it there and expect the rename followed by the `KeyMoves` move to land `plan_reviewer_fan` in `tapestry.yaml`. Otherwise card 6 must declare the rename on tapestry.

**LOW: Card 2, way-forward claim for `step`.** The sentence "so `step`, `status` and a looping step carry it in place of the generic way forward" holds for `step`.
- `internal/shedverbs/step.go:84-85` appends `armWayForward` only when the message has no `way forward:`, and the new suffix supplies one.
- I couldn't confirm the looping path, `ReportLoopArmError` in `internal/shedverbs/loopenvelope.go:215`, within the call cap. Its `loop:` rows' new suffix assertion therefore rests on an unverified assumption: that the loop path applies the same contains-check.
- `armWayForward` is only referenced in step.go, which suggests the loop path goes through the same function, but I didn't check that.

These claims hold:
- **Card 6, perchHardCap:** the template has `plan_hard_max_bounces: 0` and `review_hard_max_bounces: 4`.
- **Card 6, TestPerchHardCap:** the new local-layer row matches the existing local-file fixture form and the `--layer local` route.
- **Card 6, TestLookup:** the loom row lists the four plan open-map keys.
- **Card 18:** `TestWire_BeamRecipe` asserts EscalateJudgedFirst for each recipe and pawl's nil TraceFailure.
- **Card 10:** the tail subtest pins the edges, including Finalize's reroute to Parent-Align.

Calls made:
1. circling_test.go TestPerchHardCap and the loom template's hard-cap lines, to check the new TestPerchHardCap row and the claim that the template value is 0.
2. The non-test declaration of `plan_fan` and TestLookup's body, to check the configcli and TestLookup claims.
3. Where step appends the generic way forward, to check card 2's step claim.
4. ReportLoopArmError's way-forward handling and loom's rename table. Only the rename table came back.
5. Where ReportLoopArmError and `armWayForward` live, to check card 2's loop claim. That call was still not enough to confirm it.
