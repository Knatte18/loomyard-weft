```yaml
lens: guard-reach
outcome: reported
```

Lens guard-reach: whether each guard, refusal, branch and test row the cards add can be reached from a real entry point, and whether a guarded site recurs unguarded. Three findings, all of them guards that fire on the wrong state or on an input that can't reach them; nothing recurs unguarded.

**Per guard, reachable or not**

- Card 1 `decision-heading-malformed`, `decision-id-duplicate` and `decision-not-in-acceptance`: reachable. `TestCheckDecisionIDs` rows reach each one, and `TestNewDiscussionGate` rows cover the loom and darn gates.
- Card 1, a not-exist read of the record adding nothing: reachable whenever the record is missing, with or without the support log.
- Card 1, no `## Acceptance criteria` or no `## Decisions` section: reachable, and a row covers each.
- Card 1, the darn caller running the history check only on a clean `Validate`: reachable through the two darn gate rows (history seam called once; history seam never called).
- Card 2 skips (fence, added entry, suffixed PATTERN bullet, carry-over block, unclosed block running to the end): every state can occur in a written record. The unclosed-block case has no row, but nothing there is a refusal.
- Card 3, `ResolveGlyphs` on an empty list returning nil: not reachable from a real caller. Cards 4, 8 and 9 each call it only with at least one glyph (finding 3).
- Card 3, ambiguous and partitioned, ambiguous and not, not_found, rejected, quarry outage: reachable through `TestResolveGlyphs` rows `p#F`, `p#G`, `p#Missing`, `p#bad#x` and a missing root. `openRepo` already fails on a missing root (`repo_test.go:39-40`).
- Card 4 resolve-time `requireAbsRoot`/`requireSeam` on the loom "discussion" form: reachable in test rows only. Production `wire` and `wireLightweight` always fill both, so this is a fail-loud wiring guard and is exempt.
- Card 4 `citation-path-missing`, `citation-symbol-missing`, both `citation-symbol-ambiguous` forms, the no-commit skip (`gitrepo.ErrNoCommits` exists, `internal/gitrepo/gitrepo.go:26`) and the outage error: reachable through `TestValidateLoomDiscussion_Citations` rows.
- Card 4 guards that recur: the only callers of `ValidateLoomDiscussion` are the shedrecipe gate and `validateDiscussionCmd`. Both get a root and an index (`wiring.go:302,309`). The lightweight `decisionRecordFixture` leaves both unset, but its records carry no backticked token, so nothing reaches the index.
- Card 5 `GuardMisplaced` (seam line) and `SeamMalformed`: reachable through `TestFindSeamTests` rows.
- Card 5 stacking: no other `//lym:` placement parser conflicts. `cmd/testtiming/keep.go:15-21` already admits any `//lym:` line, and commentlint treats `//lym:` as a directive (`internal/commentlint/doc.go:21`).
- Card 6, the three `seamViolations` kinds: reachable through fixture rows.
- Card 6 real-tree row: the grep of `NumField()`, `VisibleFields(` and `NilSeams(` across test files matches exactly the tests card 6 marks, so the row can pass. The helpers `assertStructFieldsSideFree` and `assertNoZeroFields` each have one caller, the test card 6 marks.
- Card 7 `test-tier-unstated`: reachable. Its nil-`TierTags` skip is reachable through an absent `gate.yaml`, and the done-card, csharp and dispatch rows are reachable.
- Card 7, D9 recurrence: the only plan-gate builders are `planContextOf` (`entries_gate.go:126`, which serves both plan gates) and `validatePlanCmd` (`validate.go:272`). Only `loom-recipe.yaml` declares a "plan" gate, so the darn wiring needs no `TierTags` or `SeamTests`.
- Card 8 `seam-test-unlisted`, the nil-`SeamTests` skip and the reader-error return: reachable through rows. The declaring-file hold is reachable through the real index, which returns files.
- Card 8 row "`internal/c_test#TestDeps` while the test sits in package `c` passes": the matching branch is reached, but the input is probably one the index's own resolve pass rejects (finding 2).
- Card 9 `doc-go-unlisted`, and each test-code class (`_test.go` file, `_test` unit, test-only member `fixture`, rename of a test, `Test` prefix): reachable through rows.
- Card 9 holding predicate: it also fires on a state D8 excludes (finding 1).

**Findings**

1. **MEDIUM, decision — card 9 `docGoFindings` holds a card that touches no Go code.**
   - The test-code rule ends "every other ref … is non-test code", and every ref is mapped to its file unit's directory.
   - So a file target that is not a `.go` file still holds the card to that directory's `doc.go`. Examples are `internal/landingshed/template.yaml` and `internal/shuttleengine/template.yaml`, both in packages that hold a `doc.go`.
   - D8 limits the hold to targets that "touch non-test Go code" (`_lym/discussion/decision-record.md:131`), so the refusal fires on a state D8 does not guard.
   - Evidence: `_lym/plan/09-doc-go-check.md` Intent ("takes the refs … maps each to its package directory …; every other ref … is non-test code"); the `template.yaml` files are listed beside `doc.go` in their package directories.
   - Fix: say in card 9's Intent that a file target not ending `.go` is not Go code and is skipped, and add a row: a card editing only a package's non-Go file passes.

2. **LOW, consistency — card 8's `_test` pseudo-unit row likely depends on an input the upstream resolve pass rejects.**
   - The row lists `internal/c_test#TestDeps` under Uses while `TestDeps` is declared in package `c`.
   - Uses refs go through the index's resolve pass in `ValidateFormat`. A `c_test` unit that does not exist there is probably a blocking `glyph-not-found` (`internal/planglyph/resolve.go`, the not_found arm), so a real plan could not use that spelling.
   - The row "passes" only because the table counts findings by check.
   - I did not confirm how quarry answers for a missing `_test` pseudo-unit. `GoPackageCounterpart` reads no disk (`internal/planparser/targetdirs.go:338-341`), and its existing tests cover handles only, never Uses resolution.
   - Fix: have the row also assert no `glyph-not-found` for that ref, or recast it with the test in an external `c_test` file and the card naming `internal/c#TestDeps`, the counterpart direction the gate admits.

3. **NIT, design — card 3's empty-list branch of `ResolveGlyphs` cannot occur from any real caller.**
   - Cards 4, 8 and 9 each state the call is "made only when there is at least one", so "an empty glyph list returns nil and opens no repository" and its test row cover a state the callers rule out.
   - That is defensive handling for internal misuse.
   - Evidence: `_lym/plan/03-index-resolve-glyphs.md` Intent and test row; `04-citation-check.md`, `08-seam-test-check.md` and `09-doc-go-check.md` ("made only when …").
   - Fix: drop the empty-list clause and its row, or drop the callers' "only when" guards and keep one place that decides.

**Tool calls**

1. Grepped production code for `//lym:` directive parsers outside impactset, to see whether a seam line meets another placement guard; the manifest had read only impactset's parser.
2. Grepped every plan-gate builder and the recipes declaring a "plan" gate, to check whether D7's and D8's context fields recur on an unfilled path such as darn's; the manifest held only the two known builders.
3. Listed non-Go files in Go package directories that hold a `doc.go`, and Go files under `contracts/`, to test card 9's holding predicate against real inputs; the manifest had not read those directories.
4. Grepped `_test#` glyph handling in planglyph tests and `GoPackageCounterpart`'s body, to check whether card 8's pseudo-unit row survives the upstream resolve; the manifest had only the function's signature.
