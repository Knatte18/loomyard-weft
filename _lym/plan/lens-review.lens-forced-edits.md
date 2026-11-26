```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits. Findings:

1. **BLOCKING, card 4 (nested template / reviewer moves): one test fixture outside the card writes flat keys.** `internal/beamcli/dobbydeps_integration_test.go:655` writes `code_max_bounces: 7\ncode_hard_max_bounces: 9\n` into `loom.yaml`, in `internal/beamcli#TestHubLandDeps_FileTaskReleasesTheTaskAndCodeReviewCapsReadTheHubConfig`. Once card 4 nests the template, those keys become unknown, so the caps fall back to the template defaults. Once card 5 is in, the load refuses them outright. Card 4 lists `internal/beamcli#mergeParent` and `internal/beamcli#TestMergeParent` but not this test. Fix: add `internal/beamcli#TestHubLandDeps_FileTaskReleasesTheTaskAndCodeReviewCapsReadTheHubConfig` to card 4 and nest its fixture.

2. **MEDIUM, card 11 (ModelKey refusal reworded): it breaks assertions in packages the card does not list.** The text comes from `internal/shuttleengine/requires.go:60,62` ("set %s in %s.yaml to an alias…"). Three test files assert it:
   - `internal/seatengine/engine_test.go:223`, in `internal/seatengine#TestEngine_Run_StartFailureEndingTheRunStopsEveryStartedSeat`, asserts "set discussion_reviewer_model in loom.yaml to an alias…".
   - `internal/webstercli/refusedverb_helper_test.go` matches "to an alias whose engine meets them".
   - `internal/shuttleengine/requires_test.go` is covered, because card 11 lists the `shuttleengine#` package.

   Card 11 lists neither `seatengine` nor `webstercli`. The card gate runs over the card's own packages only, so these failures first show at the batch or webster gate. Fix: add those test members to card 11.

3. **MEDIUM, card 11 / D3: two production messages that name a config key bare are not on any card.**
   - `internal/burlerengine/fix.go:400` reads "the twill fixer's fan from %s needs forks, which the engine of %s cannot run". It renders `opts.FixFanKey` bare, and `internal/burlerengine/fix_test.go:413` (`TestEngine_Fix_Outcomes`) asserts that text.
   - `internal/shedadapters/burlerfix.go:261` reads "the row's fixer fan comes from %s". `internal/shedadapters/burlerfix_test.go:127` asserts it.

   The keys reach these messages through `FixFanKey`/`ReviewKey` strings built in `loomengine`, and card 4 changes those builders. The messages will then print `discussion.fixer_fan` with no module. D3 wants `loom.discussion.fixer_fan`. Fix: add `burlerengine` and `shedadapters` to card 10, or have the module-qualified key passed in.

4. **MEDIUM, card 10 (configengine's `ValidateLocal`/`ValidateMachine` and configreg's refusals reworded for every module): it misses tests in three packages.** These assert the current refusal texts:
   - `internal/shedbuild/build_test.go`, which asserts configreg's `CheckSettingKey` "is not a key of module" text. Card 4 lists `shedbuild#`, but card 10 rewords the text, so the test must be on card 10 too.
   - `internal/envconfig/config_test.go`
   - `internal/gateslot/config_test.go`

   Card 10's Edit list holds only `configengine_test#TestValidateLocal` and `configreg#TestPrepareRunSettings`. Its Intent's "a grep for each old text finds them" leaves the set open, and the plan rule asks for every asserting test to be among the card's targets. Fix: name those test members on card 10.

5. **LOW, card 4 / D3: Go comments outside the listed packages still name flat keys.** PATTERN-config-key-names covers comments.
   - `internal/shedadapters/bouncer.go:238` names `review_amend_max_bounces`.
   - `contracts/stencils/stencils.go:155,161` name `parts_depth`.
   - `internal/landingshed/prreviewask.go:33` names `pr_review_parent_timeout_min`. Card 11 lists `landingshed#` for messages only.

   No card names `shedadapters` or `contracts/stencils/stencils.go`.

6. **LOW, card 4: a reflective test may need listing.** Card 4 rewrites `loomengine.Config` and `TapestryConfig` as nested structs. Any `//lym:seam internal/loomengine#Config` test outside loomengine, such as `TestConfigTemplate_ContainsEveryConfigYAMLTag`, falls under the package glyph only if it lives in loomengine. Not verified: the tool-call budget ran out. Re-check `seam-test-unlisted` for the new nested area types, since every nested struct a reflective test walks needs its own `//lym:seam` line.

7. **LOW, card 6: the moved lines reach only part of the reconcile output.** `Result` gains `Moved`, and card 6 lists `reconcileEnvelope`. But `internal/configcli#TestConfigReconcileInGitRepo` sits on card 4, not card 6, and it may pin the envelope's keys exactly. Listing it on card 6, or under its Uses, keeps the envelope change and its assertion on one card.

Tool calls:
- Grep 1: callers of `KeyAtLine`, `reviewbudget` members, `boardCriticalSection`, and Config field readers outside loomengine. My context had only the callers the subagents sampled.
- Grep 2: flat loom/tapestry key names in files of packages the cards don't name. My context lacked a full census.
- Grep 3: the matching lines in those files and their enclosing functions, to tell real forced edits from generic fixtures.
- Grep 4: the reconcile callers, shuttleparity's reads, the enclosing test names, and the `requires.go` texts. These were needed to confirm the re-sign callers and the asserting tests.
- `lym quarry resolve` over the cards' Edit glyphs, plus the test package clauses, to check the glyphs and the `Create` handle packages. My context lacked quarry answers for these members.
- Grep 5: tests outside configengine/configreg asserting the refusal texts cards 10 and 11 reword, and the burler/shedadapters texts. My context lacked that set.
