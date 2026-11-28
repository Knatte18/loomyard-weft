```yaml
lens: code-tests
outcome: reported
```

**Behaviour → pinning test**
1. `chainWorktrees` walk (member→beam→prime, gone parent, cycle, prime alone, read error) → `TestChainWorktrees` (internal/hubgeom/nearestlive_test.go:291).
2. `OrchChain` (formats names from the walk, prime's name for `""`, no-shortname warning/nil) → reached only end-to-end through `TestPauseAndResume_ApplyTheCallerRules` member/beam rows; no row covers the no-shortname case. **Partial.**
3. `decisionOrchChain` (project record keeps the beam's own orch; task record drops its own worktree's orch) → **none**; every decision and parent-review test swaps `orchChain` for a fake (decision_test.go:615-620, parentreview_test.go:794).
4. `decisionCallerRefusal` admits an orch on the chain while the prime resolves as parent → `TestDecisionVerb_AppendsAndCommits` row "a beam's orch on the run's chain…" (decision_test.go:637).
5. `decisionCallerRefusal` chain-error with sampler-error path (now from `orchChain()`) → the fake returns `parentErr` for both, so existing rows still reach it; no distinct row for the new ordering. **Weak.**
6. `decisionCallerRefusal`: caller off the chain, then `parent()` fails → `decisionParentUnresolved` → **none** (the fake returns the same error from both resolvers, so the `parent()`-error branch at decision.go:798 is unreachable in isolation).
7. `decisionStoreVerb` admits a chain orch → `BeamOrchOnTheChain_StoresWhileThePrimeResolvesAsTheParent` (decision_test.go:1838).
8. `decisionStoreVerb` off-chain + `parent()` error → **none** (same single-error fake).
9. `parentAnswerCallerRefusal` admits a chain orch → `TestParentReviewAnswerVerb` row "a beam's orch on the run's chain answers…" (parentreview_test.go:805); chain error refuses → changed row at parentreview_test.go:157.
10. `ParentOrchName` for every parent shape → `TestParentOf` `wantOrch` column (parentof_integration_test.go:422-429). No-shortname case → **none**.
11. Batten wiring of `ParentName` into tackler/arbiter prompts (`loadedTacklerDeps`, `loadedArbiterDeps`) → **none**; `TestBattenDriverDeps` passes `"tst:orch"` to `tacklerDepsFor` but never asserts `tackler.Prompt.ParentName` (driverdeps_test.go:148).
12. Loom wiring of `c.parentName` into tackler (drivewiring.go:235) and arbiter (arbiter.go:171) prompts → **none**; wiring_beam_test.go:91-96 calls `arbiterDepsFor` but checks only record paths.
13. `HandBack` named → hand-back variant; empty → no-parent variant → `TestHandBack` (parentdirective_test.go:85).
14. `HandBack` missing `parent-directive-handback` stencil → error naming it → **none**; `TestDirective_MissingStencilNamesIt` lists only none/parent/operator-ban (parentdirective_test.go:118-121).
15. `RenderTacklerPrompt` / `shedarbiter.Prompt` use `in.ParentName` → `TestTacklerPrompt` new rows (tacklerprompt_test.go:1092-1101), `TestPrompt` (prompt_test.go:980, 995).
16. Tender `callerOf` beam-orch kind, `refuseCaller` beam-orch: pause/revive/resume of member and beam succeed; hub-wide pause/revive, outside pair, reap refused → `TestPauseAndResume_ApplyTheCallerRules` (cli_test.go:1329-1364).
17. `pairUnderBeam`: absent pair → under no beam; resolve/read error → refusal with "re-run" → **none**.
18. `refuseCaller` slugless orch name of another shortname / beam branch with empty slug → empty slug covered by the hub-wide row; other-shortname case no row.
19. `refuseReapCaller` new text and beam-orch refusal → beam-orch reap row asserts only the substring "ask your parent session" (cli_test.go:1344-1347), which both old and new texts hold.

**Findings**

- **F1 — MEDIUM, design — `internal/loomcli/decision.go:1893` (`decisionOrchChain`)** has no test: the project-record-keeps-the-beam's-own-orch / task-record-drops-its-own-worktree split, which is what admits the beam orch on its project record (#236) and keeps a task pair off its own chain, is untested. Evidence: the fakes replace `orchChain` wholesale (decision_test.go:615-620; parentreview_test.go:794); grep for `decisionOrchChain` and `OrchChain(` in test files finds no test.
- **F2 — LOW, design — `internal/battencli/drive.go:497,571` and `internal/loomcli/drivewiring.go:235`, `arbiter.go:171`**: no test asserts the parent name reaches `TacklerPromptInput.ParentName` / `PromptInput.ParentName`; dropping the field again would revert #231 with every test green. Evidence: driverdeps_test.go:148 passes `"tst:orch"` with no assertion; wiring_beam_test.go:95 checks record paths only.
- **F3 — LOW, design — `internal/tendercli/cli.go:1216` (`pairUnderBeam`)**: the absent-pair (false) case and a `ResolveWorktree`/`OrchChain` error surfacing as the "re-run the verb" refusal have no row. Evidence: `TestPauseAndResume_ApplyTheCallerRules` (cli_test.go:1329-1364) drives only present pairs.
- **F4 — LOW, design — `internal/loomcli/decision_test.go:615-620`**: the fake returns `f.parentErr` from both `parent` and `orchChain`, so the new branches where the chain resolves but `parent()` fails (decision.go:798, 1320) have no reaching row; the refusal-way-forward rule asks for a test reaching each refusal (the message is shared with a reached one).
- **F5 — LOW, design — `internal/parentdirective/parentdirective.go:860` (`HandBack`)**: the missing-stencil error naming `parent-directive-handback` has no row. Evidence: `TestDirective_MissingStencilNamesIt` rows at parentdirective_test.go:118-121.
- **F6 — LOW, design — test asserts static stencil prose**: `TestHandBack` pins the whole hand-back stencil text (`want` at parentdirective_test.go:896-897); the rubric's test-economy rule makes a new assertion on a stencil's static prose a finding — it should assert only that the marker value appears.
- **F7 — NIT, design — tautological**: `TestHandBack`'s no-parent check compares `HandBack(dir, "")` against `Directive(dir, "", false)` (parentdirective_test.go:905), both through the same `readStripped` helper; same for the `HandBack`-derived `directive` containment checks in tacklerprompt_test.go:136 and prompt_test.go:126. Assert the literal "No parent is recorded for this run." instead.
- **F8 — NIT, design — `internal/tendercli/cli_test.go:1344-1347`**: the reap row's substring check passes for the old agent reap text too, so the new reap refusal message is unpinned.

Tool calls:
1. grep of test files for `decisionOrchChain`, `OrchChain(`, parent-name wiring, and parentdirective tests — context lacked whether any test reaches them.
2. read of `TestDirective_MissingStencilNamesIt`, reap-text test mentions and wiring_beam_test assertions — context lacked their bodies.
