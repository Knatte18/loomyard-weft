```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change, everything outside the diff that reads, calls or describes it, and whether the diff leaves any of it wrong.

**Per-change affected set**

- **`runfsm.Order` removed** (`internal/runfsm/state.go`).
  - No Go caller, doc, stencil or spec names `Order()` or `runfsm.Order` any more. The search of the repo and `_bolt` records finds only a historical gate-run log under `_bolt/.lym/gate/runs/`.
  - The guard's allowlist entry in `cmd/lym/testonlycode_linux_test.go:58` is removed with it. That is required, because PATTERN-test-only-code says a stale entry fails the guard. Covered.
  - `Derive`'s doc comment used to say "first state of Order". It is rewritten to "highest-precedence state", which matches `internal/runfsm/doc.go:5`. Covered.
  - The precedence that the `Order` subtest pinned is still pinned by `TestDerive`'s "XOverY" rows (`state_test.go`). Pairs with no row (Cancelled/Cancelling, Starting/Queued) can never hold together under the conditions in `state.go:68-102`. The commit message's claim holds. Covered.
  - No board entry or plan relies on `Order`; the allowlist's "that card" refers to none found. Covered.
- **burlerengine `fakeShuttle`/`fakeHandle` trimmed.** `stopCalls`, `stoppedGUIDs`, `stop(guid)`, `blocked` and `settled` have no remaining reference. The same-named `stoppedGUIDs` in sheddriver and shedarbiter are separate fakes. `fakeHandle.Stop`'s doc comment is updated to drop "records the call". The `returned`, `stopped` and `stopOnce` fields are still used at `engine_test.go:208,233,238`. The `slices` import is dropped. Covered.
- **shedadapters `stopLog`, `probedHandle` and `withRubric` removed.** No reference remains. The `sync` import is dropped, and `shuttleengine` is still used (`attachprobe_test.go:51`). `rubricBody` is still set at `bouncer_fixture_test.go:59,108-111`. The `attachprobe_test.go` header doesn't name the removed types. Covered.
- **fabricengine `pairPermittedRoots` and `reconcilePairFor` removed.** No reference remains in the package. hubgeom has its own `reconcilePairFor` (`geometry_integration_test.go:310`), which is unaffected. `hubRelative` is still used (`livestate_matrix_test.go:223`), as is `filepath` (`reconcile_stale_registration_test.go:61`). Covered.
- **loomshed `testGateTiers` removed.** No reference remains in loomshed; websterengine has a separate constant (`template_test.go:334`). The `fmt` import is dropped. Covered.
- **websterengine `failOutput` removed.** No reference remains, and `strings` is still used (`triage_test.go:106`). Covered.
- **orchcli `sendTurn` removed.** `turnEnds` and `waitFor` still have callers (`smoke_live_test.go:233+`, `smoke_cycle_test.go`), so nothing else became unreachable. Covered.
- **reedengine `answerSession` removed.** `encodeLivePanes`, `answerFormat` and `paneGenerationFormat` still have callers (`listreconcile_test.go:181`, `namerepair_test.go:69`). Covered.
- **reportengine `commitOffChain` removed.** `commitOffChainOn` still has callers (`fake_test.go:510,582,603`). Covered.
- **battenshed `resumeFrom` removed.** No reference remains. The `loomcli` `resumeFromRunState` is a different symbol. Covered.
- **shuttleengine `skillStartFixture` removed and the doc comment moved onto `skillStartFixtureWith`.** Its callers (`run_test.go:741`, `startup_test.go:1632,1923`) are unchanged. The new doc comment no longer leans on the removed wrapper. Covered.
- **Change description (`_lym/landing/summary.md`).** I checked the per-package net-line claims against `git diff --numstat`. All match except shuttleengine (see finding 1).
- **Rubric checks:**
  - Declared PATTERN changes: the directive declares none, so there is nothing to land.
  - Commit messages: there are no `Pattern-Departure` or `Loosened-Check` trailers, and the claims in the messages hold as above.
  - Stencil wording: no stencil is touched.

**Findings**

1. LOW · consistency · `_lym/landing/summary.md:26` (the `internal/shuttleengine` row of the Removed table). The row claims 6 net lines removed, but the diff removes 5 net. Evidence: `git diff --numstat` shows `3 8 internal/shuttleengine/startup_test.go`, so 8 − 3 = 5.

Tool calls:
1. A grep of repo Go, md and yaml files and `_bolt/_lym/specs` for `Order()`, `runfsm.Order` and the old doc phrases, plus the headers and imports of the changed files and `runfsm/doc.go`. The manifest had no doc or spec search, and didn't cover import use after the deletions.
2. A grep of the `_bolt` records for planned uses of `runfsm.Order`. The removed allowlist entry referred to a future card, and the manifest didn't cover the board records.
3. `git diff --numstat` over the review range. Checking the summary's per-package line counts needs exact numbers, which the diff pieces don't give.
