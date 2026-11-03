```yaml
lens: fix-claims
outcome: reported
```

Checked every claim the fix added or rewrote against the code at HEAD.

**MEDIUM** — `_lym/plan/00-overview.md`, Shared Decision `d3-sites-outside-fabric`, the new sentence that says `internal/adoptcli/adopt.go`, `internal/cleanupcli/reconcile.go` and `internal/hibernateengine/wake.go` "quote the same fabric text only in report lines … and the verb printing them goes on; they are neither a stop nor a refusal."
- **Summary:** two of the three sites are refusals that stop their verb, so the reason the decision gives for exempting them is false.
- **Evidence:**
  - Both `sharedStopText` functions (`internal/adoptcli/adopt.go:670`, `internal/cleanupcli/reconcile.go:81`) feed refusals.
  - `internal/adoptcli/adopt.go:239-240`: `return finish(), fmt.Errorf("lym adopt: the origin reconcile stopped, so nothing ran: %s", stops)`.
  - `internal/cleanupcli/cli.go:100-101`: `return fail("cleanup --apply: the origin reconcile stopped, so no segment ran: " + stops)`.
- **What holds instead:** both `sharedStopText` functions keep only shared branches (`branch.Repo == sideBolt || (branch.Repo == sideRecords && branch.Prime)`). A slug pair's gone-branch text naming `lym fabric remove <slug>` therefore never reaches these refusals. The exemption is only sound on that ground.
- **Other parts of the sentence:**
  - The `internal/adoptcli/adopt.go:553` warning line does fit "report line, verb goes on".
  - `internal/hibernateengine/wake.go`'s `runStep` (`:128-145`) builds a wake's step list, "X stopped: <way forward>; then …", which is closer to a stop's way forward than a report line. That part of the claim is arguable.

The other claims hold:
- **Card 6:** the callers are in package `fabricengine_test`, so they cannot read `weftArchive`'s unexported fields.
- **Card 5:** `SetTeardownWaitForTest` sets the wait per Topology (`internal/fabricengine/export_test.go:698-701`).
- **Card 8:** the live-pair skip `continue`s with no entry (`internal/fabricengine/cleanup.go`, about line 200). `afterEnumerate` exists in `cleanupRemoteWarp`.
- **Card 10:** `armAt` is called with verb `run` from `arm`, `ArmAt` and `StartFirstPass` (`internal/battencli/arm.go:345,569`, `internal/battencli/cli.go:637`). Its `hubOwnerSeams` and `hubStartGuard` defaults sit at `internal/battencli/arm.go:379-384`.
- **Card 13, landingshed:** no landingshed test calls `stoppedReason`, `reportVerdict` or `reconcileVerdict` directly, so "callers on this card" holds. The callers are `ParentAlign.Call` and `Finalize.ownerChecks`.
- **Card 13, envkit:** `nilLegal` holds `"Landing.Reconcile"` (`internal/testkit/envkit/envkit.go:69`).
- **Card 13, cancel:** `cancelDeps.ReadStatus` exists (`internal/battencli/cancelcmd.go:220`), and `cancelFake` has `status` and `statusFound`.

Tool calls:
1. Grepped the landingshed tests for direct calls to the verdict helpers, and adoptcli, cleanupcli and wake for the D3 site claim. Context lacked those call sites.
2. Read the adopt warning and stop blocks and found the `sharedStopText` callers in adoptcli and cleanupcli, to check whether those verbs go on. Context lacked them.
3. Read the `sharedStopText` call sites in `internal/cleanupcli/cli.go` and `internal/adoptcli/adopt.go`, and the wake step assembly. This confirmed the refusals.
4. Read `sharedStopText`'s shared-branch filter, `Cleanup`'s live-pair loop, `cancelFake`'s fields and loomcli's imports, to check the remaining claims. Context lacked them.
