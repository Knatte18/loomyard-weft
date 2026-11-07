```yaml
lens: code-tests
outcome: reported
```

Task: list, per behaviour the diff adds or changes, the test that pins it (lens: code-tests).

**Per-behaviour list**

1. `beamTemplate` build yields marked beam "b", member "m" (parent b) and plain pair "p", unpushed.
   - Pinned by `TestResolveBattenPrime` (`internal/battencli/beamguard_integration_test.go:68-79`).
   - Its rows fail if the beam mark, the member's parent or the plain pair is wrong:
     - `MarkedBeamReturnsThePrime` needs "b" marked.
     - `PairForkedFromTheBeamNamesTheBeam` needs "m" parented to "b".
     - `PairForkedFromThePrimeNamesThePrime` needs "p" parented to the prime.
   - Also pinned by `TestVerbs_RunFromABeam` (lines 186-230).
   - The `SkipPush` behaviour has no assertion of its own. Neither does the template-copy mechanism, whose test is `hubforge/template_test.go`'s `TestCopyRegistered_CopyHoldsTheBuildAndIsolatesWrites`. Both are fixture mechanics, not behaviour this diff adds.

2. `beamHub` returns a copy of the registered template plus the prime name.
   - Pinned by every caller's use of `primeName`:
     - `TestResolveBattenPrime` line 87.
     - `TestVerbs_RunFromABeam` line 198.
     - `TestCancelCmd_RemoveRefusesFromInsideTheBeam` line 242.

3. Copy isolation: each test mutates its own copy.
   - `TestResolveBattenPrime` mutates the board via `SetStatus` and `UpsertTask`.
   - `TestVerbs_RunFromABeam` commits a finished run.
   - Both would break the other tests in the same binary if the copy shared state with the template.
   - Pinned by those tests, run in parallel, and by `hubforge`'s `TestCopyRegistered_CopyHoldsTheBuildAndIsolatesWrites`.

4. The copy is taken before `boardkit.InstallReplay` in `TestResolveBattenPrime` and `TestVerbs_RunFromABeam` (diff, lines 58-62 and 163-167).
   - This is an ordering property. The tests cannot fail on it if it is reversed, because the template build is a once-only effect that happens whichever test reaches it first.
   - The `hubforge` template-build guards cover the replay hazard: `Build`'s doc forbids code whose result a Bolt replay changes.
   - This is a review-time criterion in the decision record, "Review: each test that installs a process-global seam takes its copies before installing it". No test is meant to pin it.
   - Not a finding.

5. `TestCancelCmd_RemoveRefusesFromInsideTheBeam` installs no replay, so it has no ordering change. It is pinned by its own assertion.

**Findings**

None. No behaviour the diff adds or changes lacks a pinning test, and the diff adds no new tests, which fits test economy.

**Tool calls**

None; the inherited context held everything needed.
