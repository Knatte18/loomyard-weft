```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set — the set each change forces outside its own lines**

| Change | Set found | Covered |
|---|---|---|
| `composeReviewPrompt` gains `parentRules` | One production caller (`review.go:133`), `assist.go`, and the tests in `template_test.go`, `assist_test.go`, `parentseat_test.go` and `prompt_test.go`. | Covered. Every call passes the new argument. |
| `parent_rules` marker in `burler-step-review.md` | Optional marker passed in `prompt.go`; the header comment is updated; the stencil registry entry and `markers_test.go` are added. | Covered. |
| `ParentAsk.Seed`, `RoundAsk`, `EndParentAsk`, `SettleParentAsk` | Other readers of the ask record are `loomcli/parentreview.go` (answer and await verbs) and its test. These use `Latest`, `AnswerParentAsk` and `ExpireParentAsk`, which the diff leaves unchanged. `Seed` is `omitempty`, so records without it still parse. | Covered. |
| `ParentReviewDirName` | Defined in `lymdirs/dirs.go:70` and used only in `ParentReviewFilesOf`. | Covered under `PATTERN-lym-files`. |
| `Finding.ParentIDs` and the review-format key | `burler-review-format.md` names the key. Only `verdict.go` and the gate read it, and `verdict_test.go` has a case. | Covered. |
| `SegmentSettings.Parent` and `BurlerHalfDeps.Parent` | The constructor in `entries_burler.go` and the setter in `loomcli/wiring.go` (Discussion perch only). | Covered. |
| `concludeReview` gate reason falls back to `gate.Findings` | The error text only; a search for the old message string in tests shows no other asserter. | Covered. |
| `chairAwaitCap` extracted from `segmentAssistBounds` | Both callers (`wiring.go`) now use it. | Covered. |
| Docs | `docs/overview.md`, the `doc.go` files of `burlerengine`, `shedadapters`, `loomcli` and `orchcli`, and `orch-role.md` all describe the new behaviour. | Covered. |
| `[discussion-review request]` tag | The tag constant is in `shedadapters/parentseat.go`. The orch-role section and the doc comment use the same string. | Covered. |
| Commit messages and `Pattern-Departure:` trailers | No trailers in the three commits. | No departure to judge. |
| Declared PATTERN changes | The directive declares none, and the diff touches neither `PATTERN.md` nor `pattern/`. | Nothing to land. |

**Findings:** none.

Tool calls (one Bash call, five greps):
1. Callers of `composeReviewPrompt`, to check the signature change reached every one.
2. Uses of `ParentReviewDirName`, to check the `lymdirs` naming rule.
3. Files that read or write the ask record, to find consumers outside the diff.
4. Uses of `loomProgram`, to check the signing name.
5. Mentions of `[discussion-review request]`, `assist_rules` and `origin: parent` in docs and stencils, to find places that need the new marker or key.
