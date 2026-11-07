```yaml
lens: code-correctness
outcome: reported
```

**Task:** correctness lens over the `beamHub` and `beamTemplate` change in `internal/battencli/beamguard_integration_test.go`. I judged from the inherited context and one grep. I ran no tests, because the lym rules refuse a raw `go test`.

**Per function and branch**

- **`beamTemplate.Build`:** I ran it on the empty prime-anchored base.
  - The three `AddPairWith` calls run in the order b, m, p with the options `{SkipPush, Beam}`, `{SkipPush, Parent: "b"}` and `{SkipPush}`. That matches the removed per-test lines exactly. Correct.
  - A failing `AddPairWith` calls `tb.Fatalf` on the build TB, which records the failure and ends the build goroutine. `templateFor` then fails the caller naming the key and marks the slot failed. Correct.
  - The build calls no `Setenv`, `MkdirTemp`, global git config, seam setter or once-only registration, and it does not use `tb.Context()`. Correct.
- **`beamTemplate.Base = Shape{Anchor: "."}`:** this equals the old `CopyHub` argument, with no base pairs. Correct.
- **`beamTemplate.Key`:** `"battencli-beam"` is used once in the repo, and none of the shape names (for example `".+"`) collide with it. `slotFor`'s address check passes for the one package-level value. Correct.
- **`requireBoardAtUpstreamTip` after the build:** this is the one residual risk. Every `AddPairWith` here has `SkipPush`, so `CoalescePushBothAt` pushes nothing. I did not confirm that `Topology.Add` leaves the board's HEAD unmoved. The grep found no board commit on the `SkipPush` path, but I did not read `add.go`'s board handling. If `Add` commits on the board branch without pushing, the template build fails with "the board's HEAD ... is not the tip", and every `beamHub` caller fails. The commit message and summary report only `go vet`, not a run. This is unverified, not a found defect. The handler can settle it with `lym gate test ./internal/battencli`.
- **`beamHub`:** I ran it with a copy from `CopyRegistered(t, &beamTemplate)`.
  - `PrimeName(h.Location)` reads the copy's own location, so it returns the prime's name correctly.
  - A `PrimeName` error is a `t.Fatalf`. Correct.
- **`TestResolveBattenPrime` and `TestVerbs_RunFromABeam`:** the copy now comes before `boardkit.InstallReplay`, so the template build (first use, any test in the binary) cannot run under the replay. `t.Parallel()` comes first in both. This is only half the criterion, because another parallel test may install the replay while a different test's first use builds the template. Only tests that install the replay do so, and both now take their copy first. Even so, two parallel tests can interleave: test A installs the replay, then test B's first `CopyRegistered` builds the template under A's replay. I did not read how `InstallReplay` scopes its seam (per test or process-global). If it is process-global and unsynchronised, a race remains in principle. In practice `beamHub` is called at the start of each test and the template builds under a slot mutex. Whether that race can occur depends on `InstallReplay` and on test scheduling, so I cannot call it correct from the context I hold. I report it as LOW.
- **`TestCancelCmd_RemoveRefusesFromInsideTheBeam`:** it installs no replay and takes its copy at the start. Correct.

**Findings**

1. **LOW, class `design`.** `internal/battencli/beamguard_integration_test.go:61-62` and `:166-167`.
   - Taking the copy first does not by itself keep the template build clear of a process-global replay another parallel test has installed. The acceptance criterion says a test takes its copies before installing the seam.
   - It is guaranteed only if no other test in the binary installs the replay before the template's first build.
   - Evidence: `TestResolveBattenPrime` and `TestVerbs_RunFromABeam` both call `t.Parallel()` and then install the replay. `TestCancelCmd_RemoveRefusesFromInsideTheBeam` can build the template while either has already installed it.
   - Confirm against `boardkit.InstallReplay`'s scoping. If it is process-global, this is a defect.
2. **Unverified, not a finding:** the `requireBoardAtUpstreamTip` outcome described above.

**Tool calls**
1. `grep -rn "SkipPush" internal/fabricengine/*.go`: to check whether `Add` with `SkipPush` leaves the board HEAD at the upstream tip. The context lacked `add.go`'s board handling, and the result was inconclusive.
