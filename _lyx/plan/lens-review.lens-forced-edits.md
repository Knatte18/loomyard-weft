```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits. Findings:

1. **BLOCKING** — Card 16 (D11, `HandsBack`). The plan has `HandsBack` read `parentAlignName`, but `internal/landingshed#TestHandsBack` still has rows with producer `"Merge-In"` that want `true` (`internal/landingshed/handback_test.go:22`, `:24`). Neither card 16 nor any other card lists it, so it fails card 16's gate. Fix: add `internal/landingshed#TestHandsBack` to card 16's `**Edit:**` and rewrite those rows to `Parent-Align`.

2. **BLOCKING** — Card 16 (D11, recipe row names). `internal/loomrecipe#TestEntryNumbers` pins the embedded recipe's entry numbers as `"Merge-In": 8` (`internal/loomrecipe/recipe_test.go:351`). After the rename the recipe has no `Merge-In` row, and no card lists this test. Fix: list it on card 16 and key the entry `Parent-Align`.

3. **BLOCKING** — Card 6 (D3). `internal/webstercli#seedLoomConfigWithFriction` writes `parent_review_wait_min: 60` into a test `loom.yaml` (`internal/webstercli/wiring_test.go:83`). Card 6 drops the key from `Config` and the template, so the strict load refuses the seeded file. Card 6 lists only loomcli's helper of the same name. Fix: add `internal/webstercli#seedLoomConfigWithFriction` to card 6.

4. **MEDIUM** — Card 16 (D11, gate key). The shared env fixture still says `merge-in` in code: `GateKey: "merge-in"` at `internal/testkit/envkit/envkit.go:151`, and the gate-key tier map entry `"merge-in": {0, 1}` at `:231`. `internal/loomrecipe/fixture_test.go` and `internal/loomrecipe/escalation_test.go` read this fixture and build a recipe whose row now runs key `parent-align`. Card 17 lists `envkit.go` as Prosa ("comments only"), and no card retargets these code lines, so a recipe-level test that reaches the row's gate key finds no `parent-align` entry. Fix: list `internal/testkit/envkit#LandingDeps` and the tier map's member on card 16, renamed to `parent-align`.

5. **LOW** — Card 16 (D11, attempt-record fallback tests). The Intent says the new fallback rows go in the renamed `parentalign_fixer_test.go`. But the card names no member of that file under `**Edit:**` and no `plan:` handle under `**Create:**`, though it changes `TestMergeIn_Fixer` and the `mergeInFixture` helpers (`attemptsOnDisk`, `seedAttempts`, `fixerReportPath`). Fix: name the test functions the fallback rows extend, so the scope guard and done-checks see them.

6. **LOW** — Cards 3/4 (D2, "no text names parent review"). Two files outside every card's targets still name parent review: `internal/loomcli/resume.go` and `internal/loomcli/wiring_commitstatus_test.go`. Fix: add them to card 3's or card 4's prose edits.

7. **LOW** — Card 16 (D11). `internal/loomshed#TestDarnWrite_Call` asserts `wantReasonLacks: "Merge-In"` (`internal/loomshed/darnwrite_test.go:93`). After the rename that assertion passes whatever the reason says. Fix: change it to lack `Parent-Align`.

Calls made:
- Mapped every `Merge-In`, `NameMergeIn`, `MergeIn` and `mergeInName` reference in test files to its enclosing function, because my context had only file-level counts.
- Printed the matching lines for the test members no card lists, to tell real assertions from comments; my context lacked those lines.
- Mapped the remaining `ParentReview`, `alignment` and retired-key references to their functions, to find callers outside the deleted files that no card lists.
- Printed those lines for the unlisted members (webstercli, reportengine, shedrecipe), which my context lacked.
- Mapped the `merge-in` gate-key and `MergeFixAttempts` readers across the repo; my context had only the production sites.
- Read envkit's `merge-in` lines and listed the tests that use the fixture, to judge finding 4; my context lacked both.
