```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — for each behaviour the diff adds or changes, name the test that pins it, or none.

**Per-behaviour list**

1. `beamHubRegistration.Build` creates the beam pair `the-beam`, `member-with-pair` and `forked-child`, with the parent fork on the last two. Pinned by `TestMemberPairs_ReturnsEveryLocalPairUnderTheBeam` (`projectrecord_integration_test.go:69`). It asserts exactly `forked-child` and `member-with-pair`. It would fail if either pair or the beam were missing.

2. The uncommitted `board.json` (beam, member with a pair, member with none) survives `CopyRegistered`. Pinned by `TestMemberPairs_ReturnsEveryLocalPairUnderTheBeam` and `TestDecisionReceivers_ReachABeamsMemberPairs` (`projectrecord_integration_test.go:88`). The first expects `member-no-pair` excluded, which needs the board entries. The second expects the beam's strand plus the member pairs' strands, which needs the beam flag.

3. The shortname record `tst` survives the copy. Pinned by `TestDecisionParent_ABeamsRecordIsItsOwnOrchs` (`projectrecord_integration_test.go:120`). It expects the orch name formatted from `testShort`. Its subtest `AHubWithNoShortnameIsRefusedNamingTheWayForward` removes the file from its own copy and expects the refusal. That also pins copy isolation: removing the file in one copy does not affect the parallel `TheBeamsOwnLiveOrchIsTheParent` subtest.

4. Both board files stay uncommitted after the copy. Pinned by `TestCommitRunRecord_ProjectRecordCommitsItsDirectory` (`projectrecord_integration_test.go:153`). It commits the project record and expects the discussion record left uncommitted. This test does not directly assert that `board.json` itself is uncommitted. That property is unchanged from before, since the old code left the files uncommitted too, so I count it as covered.

5. `badReedUpRegistration.Build` corrupts the `mouse:` line, so reed Up fails. Pinned by `TestLoomFailedReedUpRefuses` (`smoke_prebootstrap_integration_test.go:260`) and `TestLoomResumeReconcilesTheHubConfigBeforeArming` (`smoke_prebootstrap_integration_test.go:195`). Both expect start or resume to refuse at reed Up.

6. `SeedConfig` map and `AddPair(badReedUpSlug)` in the template: the pair exists with the origin record committed. Pinned by `TestLoomFailedReedUpRefuses`. The `ReadOrigin` call in the wrong-parent subtest needs the committed origin, and the seeds `seedGoDriverRun` and `seedLLMDriver` need it too. The providerless config is pinned only indirectly, as the refusal path would differ with a real provider. That property is unchanged.

7. Each site's `seed` still runs on its own copy after `lymcwd.Resolve`. Pinned by the four sites in `smoke_prebootstrap_integration_test.go` (lines 200, 292, 321, 329). Each asserts a seed-dependent outcome: the retired key removed on resume, the `bootstrap` kind, the `unseeded` kind and its message. Seeds on separate copies would fail if they leaked: `go driver` and `llm driver` need different seeds, and `WriteSeed` refuses a disagreeing existing seed. They run in parallel subtests, so sharing a copy would fail.

8. A copy is isolated from the template and from other copies. Pinned by item 3's subtest, and by `hubforge`'s own `template_test.go` (`internal/hubforge`). I did not open it, because registration mechanics belong to the `hubforge` task, not this diff.

**Findings**

None. No behaviour in the diff lacks a pinning test, and no new test was added, so there is no redundant or duplicate test and no test-economy finding. The diff changes fixture construction only, and every pre-existing assertion is kept.

One note, not a finding: no loomcli test pins the registration keys' uniqueness. `hubforge`'s `slotFor` fails any duplicate at run time and its own tests cover that, so I raise nothing.

**Tool calls**

No tool calls were made. The inherited context already held every file this lens needed.
