```yaml
lens: code-scope
outcome: reported
```

Task: map every hunk of the diff to the decision that builds it, and every decision to its builders, for the code-scope lens.

**Change → decision**
- `internal/fabricengine/worktreetree.go` and `fabricengine/doc.go`: D5 (card 1). `ReviewTreeID`/`ReviewTreeChanges` closures in `loomcli/wiring.go` and `battencli/wire.go`, `shedrecipe/recipe.go` `Env` fields, `envkit.go` stubs: forced by D5 and the PATTERN-fabric-boundary constraint (card 12). The `destructiveguard_test.go` allowlist entry is forced by `WorktreeTreeID`'s `os.RemoveAll`.
- `internal/shedadapters/targetsnapshot.go`, `lymdirs.ReviewSnapshotDirName`, `shedrecipe.reviewTargetSnapshot`, and the `shedrecipe/seam_enforcement_test.go` `lymdirs` import allowance: D5 (card 3). The seam-test edit is forced by the new import.
- `roundsnapshot.go`, `roundSnapshotPath`, `BurlerReviewProducer.recordRoundStart`, and `BurlerFixProducer`'s `decisions_routed` write: D5 (card 4).
- `burlerengine/fixerreport.go`, the `fix.go` `spawned: false` frontmatter and gate entry, and the `burler-step-fix.md` stencil: D1 (card 2).
- `burlerengine/roundcheck.go`, the `verdict.go` `prior` and `carries` additions, `review.go` gate wiring, and the `burler-review-format.md` additions: D3 and D4 (card 5).
- `burlerengine/prompt.go` prior-findings block, the `profile.go` `Round` field, and the `burler-step-review.md` rules: D2 (card 6).
- `shedadapters/roundcontext.go` (`BuildRoundContext`, the changes file, `withParentReportPath`), and `burlerreview.go` `roundProfile` and `recordRoundChanges`: D2, D3 and D4 (card 7).
- `burlercli/validate.go` and its help text, `cli.go` and `doc.go`: D3 (card 8).
- `Bouncer.noticeWayForward` and `renderEscalation`: D2 and D6 (card 9). Card 9 licenses the clause on every cause's notice, so appending it to circling, budget and hard-cap notices is not unrequested.
- `bouncerstalemate.go`, `settleUnlessRefused`'s `recordPostFixTargetID`, `settleUndecided`'s stalemate branch, `EscalationStalemate` with its validity messages, and `BouncerConfig` `StalemateRounds` and `Snapshot`: D6 and the arbiter's yield entry, which `stalemated` implements through `designWrongKeys` (card 10).
- `arbiterstop.go`, `loomcli/circling.go` and its text, `loomcli/doc.go`, `arbiterstop/doc.go`, the `loom-template-arbiter.md` and `orch-role.md` stencils, the `bouncer-template-escalation.md` cause line, `docs/overview.md`, and the `config.go`, `template.yaml` and `ConfigSpec` `review_stalemate_rounds` entries: D6 (cards 11 and 12). The docs and stencil edits are forced by the new cause.
- `pattern/PATTERN-sole-parsers.md`: the declared constraint, which adds a background section and one bullet to the entry's background list. The entry's own line in `PATTERN.md` is untouched in this diff.

**Decision → builders**
- D1: card 2. D2: cards 6, 7 and 9. D3: cards 5, 7 and 8. D4: cards 5 and 7. D5: cards 1, 3 and 4. D6: cards 9 to 12. Arbiter yield entry: card 10. Every decision has a builder.

**Findings**
None. Every hunk traces to a decision or is forced by a traced change, and every decision has a builder.

Tool calls:
1. Read `_lym/plan/09-escalation-notice-way-forward.md`, the `docs/overview.md`, `destructiveguard_test.go`, `envkit.go` and `seam_enforcement_test.go` diffs, and a grep of card 12: my context lacked the card text and these four hunks, and I needed them to judge whether the all-cause notice clause and the wiring edits were licensed.
