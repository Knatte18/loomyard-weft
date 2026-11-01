```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. I re-ran the set searches for the findings whose premise is a set (F1, F3, F5, F6, F7, F8, F9), most severe first.

Findings I left unchecked: F2, F4, F10, F11, F12 and F13. None of them has a set premise: F2 and F3 are decision findings, the parent ruled on them, and I treated the rest as single-site.

Results of the re-run searches:
- **F1 (`PartsMax`):** covered. The only `PartsMax` left in the plan is `loomengine.Config.PartsMax` in cards 1 and 13, which is a separate field and not an Env field.
- **F3 (respell clause and bullets):** covered. No respell wording remains in the cards.
- **F5 (pointers to the format's home):** covered. The fix reaches every pointer in specs, docs and the rubric. The only other hits are `docs/research/*/data/prompts/*.md` prompt dumps, which are measurement records and out of scope.
- **F7 (reportengine plan-phase list):** covered. `internal/reportengine/doc.go:130` is the only prose list of plan-phase rows.
- **F8 (planglyph doc):** covered, since both mentions of `caller-uncovered` are reworded.

Findings:

- **LOW**, card 9 Intent, the reworded `Table.Validate` and `Seat.Accept` doc comments and the `doc.go` rewording. The fix covers `doc.go` lines 26, 61 and 87 and the `Required`/`Accept`/`Table.Validate` lead-directive comments. It misses other sentences that writer seats also make false:
  - `internal/seatengine/table.go:140`: the `Table.Validate` comment's "…is neither Required nor given an Accept hook; Accept belongs to a Required advisor."
  - `table.go:179`: the `validateSeatRoles` comment's "Accept only on a Required advisor". Card 9 edits that function's rule, but this comment is left as is.
  - `table.go:115`: the `Seats` field's "then the named advisors".
  - `table.go:22`: the `chairDirectiveBlock` comment says a lead with more than one seat includes `seat-directive-chair`, but a writer table's lead includes the writer chair block instead.
  - `internal/seatengine/doc.go:24`: "every later seat is an advisor with a unique valid name".

  Evidence: the grep output above.

- **LOW**, card 7, the new `loom-plan-spec.md` sentence about the modes the context checks run in. That sentence covers the mode list at line 414. But the per-check rows say only `loomshed.ValidatePlan` and `loomshed.ValidateReworkPlan` "append it with the context checks", and `ValidatePart` now also appends them through `planContextFindings`. These rows go stale:
  - `contracts/specs/loom-plan-spec.md:631` (`test-tier-unstated`)
  - `:640` (`seam-test-unlisted`)
  - `:654` (`doc-go-unlisted`)

  Evidence: the grep output for those lines, and card 7's text "`planContextFindings` over every card with pc's `DecisionRecordPath` cleared".

Tool calls:
1. Grepped specs, docs and plan cards for the F5, F1 and F3 sets, which my context showed only by the review's line numbers.
2. Grepped `internal/seatengine/doc.go` and `table.go` for advisor-only wording (the F9 set), which my context lacked beyond the review's named lines.
3. Grepped the `loom-plan-spec.md` mode and context-check sentences and the reportengine and overview phase lists (the F6 and F7 sets), which my context lacked beyond line 414 and line 130.
