```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged each reviewer proposal the fixer carried out as proposed (F1, F3–F6, F8–F11) and the parts of F2 taken as proposed.

**1. MEDIUM — card 13, the new landingshed bullet (`Deps.RemoveFollowUp`, filled by `landingDeps` with `pairteardown.BattenFollowUp`)**

Summary: in Finalize's parent-already-holds-the-work path, the follow-up names `lym batten cancel <slug> --remove` for work that has already reached the parent.

Evidence:
- `BattenFollowUp` decides whether the work has landed from the batten record's row alone.
- The landing producers run while the batten record is still at Run-Shed. `battenrecipe/names.go:43` says `NameLand` is the later row "that lands a finished child into the prime: pull, deploy, board prune…".
- `internal/landingshed/finalize_owner.go:36-37` runs `reconcileVerdict(ctx, finalizeName, fz.deps.Reconcile)` exactly when `ParentHoldsTaskWork()` is true.
- Concrete case: an earlier Finalize merged and pushed the parent, then stopped before marking. The re-entry finds the weft branch gone (`PushFailureGone` way forward naming `lym fabric remove`) and no owner pause. The record is at Run-Shed, so the new stop says to cancel with `--remove`.
- D2 treats that as releasing a landed entry, which is the outcome the Land branch of `BattenFollowUp` exists to forbid.
- The card should leave this path without a follow-up, or pass a value that reads as landed. For example, `reconcileVerdict` on the held path could pass nil for `followUp`, as the adopter-first exception does.

**2. LOW — card 10 Intent, "A receiver whose `readopt.Add` is nil, every hand-built test receiver and the step verb's receiver among them, keeps today's path" (taken from F4's proposal)**

Summary: once `armAt` fills `readopt` for verb run, a hand-built receiver that calls `armAt`/`arm` with "run" carries the real `hubReadoptSeams`, so the sentence is wrong for those receivers.

Evidence:
- `internal/battencli/startguard_integration_test.go:53` calls `c.armAt(h.Location, slug, true, "run")`.
- `startguard_integration_test.go:78` and `lifecycle_integration_test.go:264` call `(&battenCLI{}).arm(pair, "run", …)`.
- Each builds the receiver by hand and goes through `armAt`.
- The sentence should say "every hand-built receiver that calls no run-verb arm".

I found no case against F1, F3, F5, F6, F8, F9, F10 or F11.

Tool calls:
- `grep internal/battenrecipe` for `NameLand`: to learn which shed row the landing producers run under, which my context lacked.
- `grep` battencli/shedcli tests for `armAt`/`Arm` with "run": to find hand-built receivers that reach `armAt`, which my context lacked.
