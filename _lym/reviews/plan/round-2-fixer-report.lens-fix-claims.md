```yaml
lens: fix-claims
outcome: reported
```

Checked the claims the fix added or rewrote against the code at HEAD.

**F1. MEDIUM — the `envelopeOf` slugs-list claim breaks on early stops**
- **Changed text:** card 6, the new runSteps sentence "`envelopeOf` (card 7) names the `Subjects` without `root` to `collect`, so the envelope's `slugs` list keeps today's members". Card 7's matching `envelopeOf` sentence makes the same claim.
- **Summary:** `collect` over the `Subjects` keeps today's members only if every named subject has an outcome. On a refused reconcile, no part carries one.
- **Evidence:**
  - `collect` dereferences `*outcomes[slug]` for every named slug (`internal/adoptcli/adopt.go:1038`).
  - Card 6 seeds outcomes in `runSteps`' memory, not in any step's part. Card 6 says a part holds "only the `slugOutcome` fields that step set".
  - A reconcile refusal sets no slug field, so `envelopeOf` over a record stopped at `reconcile` has no merged outcome for the named slug. It either panics or must seed the slug. A seeded slug is a `slugs` member that today's reconcile refusal never carries: `finish()` at `adopt.go:258` leaves `result.Slugs` unset.
  - The new `ReconcileErrorRefuses` envelope assertion in card 7 reaches exactly this path.
  - The fix states neither the seed nor the narrower guarantee.

**F2. LOW — the overview's `reentry` carve-out names only refusals**
- **Changed text:** `00-overview.md`, Shared Decisions: "the existing reentry text of the takeover's own refusals (`reentry`) stays".
- **Summary:** `reentry` also ends way-forward warnings, which the kept TestAdopt rows assert. The carve-out names refusals only, and the bullet's first clause still covers "every ... way forward".
- **Evidence:**
  - `reentry` ends each reconcile warning (`adopt.go:592`) and each shared-stop line (`adopt.go:732`).
  - The kept rows `RewrittenPrimeBranchWithNoWayForwardIsAWarningNamingFabricPull` and `DivergedBranchOfAnAdoptedBeamCarriesItsWayForwardInParentheses` (`adopt_test.go:676`, `:683`) assert those warnings.
  - Card 6 moves `reconcileWarnings` into `stepOwner`, so a reader may take those warnings for "a way forward the worker adds" and switch them to `Request.Command`.

**F3. LOW — one comment clause in card 6 is a little looser than the code**
- **Changed text:** card 6 Intent, "`runBeamSteps`' and `heldBeamWayForward`'s docs, since the landing resumes by hand with `lym tender resume` and `lym loom resume`".
- **Summary:** card 6's own step 6 names `lym tender resume` only when `paused` is set, and `lym loom resume` alone otherwise. The comment wording "with `lym tender resume` and `lym loom resume`" claims both every time.
- **Evidence:** card 6, step 6: "then it names `lym tender resume <slug>` and `lym loom resume` in its pair, and otherwise `lym loom resume` in its pair alone".

**Claims that held**
- Today's refusals carry the reconcile report, the mutation record and the deploy outcome: `adopt.go:255-258`, `:362-365`, and the test at `adopt_test.go:937`.
- `root` gets no outcome today.
- `reconcileWarnings` needs `adopted` and `Stopped` (`adopt.go:566`, `:578`).
- The deploy warning follows the reconcile warnings (`adopt.go:413-416`).
- The steps up to `owner` share one process (Shared Decisions).
- The world's `Reconcile` fake returns `w.report` beside `w.reconcileErr` (`adopt_test.go:140`), so mutations can come back beside the error.
- `lymdirs.AdoptDirName` is the declarer `fabricengine.AdoptScratchDir` joins (`internal/fabricengine/junctionnames.go:192`), and its comment reads as F7 says (`internal/lymdirs/dirs.go:40`).
- `TestFollow` pins the interrupted text, and `TestAdoptSectionOf` pins the stopped way forward.
- `battencli` copies `coverage.Held` (`dobbyreads.go:168`).

**Tool calls**
- The parent's earlier Bash call: one grep over `internal/adoptcli` (`reentry` uses), `adopt_test.go` (the world's `Reconcile` fake) and `internal` (the `AdoptDirName` declarer). My context lacked the `reentry` call sites outside `adopt.go`, the fake's return, and the code that declares and uses the adopt directory name.
