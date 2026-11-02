```yaml
lens: fix-affected-set
outcome: reported
```

Task: re-run each set-premised finding's search and check the fix covers every member.

Unchecked findings: none. I checked F1, F2, F3 and F4, plus the bare card-number mentions that `insert-card` reported after the renumber.

no findings

- **F2:** every remaining comment that names a member card 3 deletes now has a target.
  - The `render.go`, `strand.go` and `recoverbatch.go` file headers are on card 4.
  - The `testReplyPath` doc comment is on card 3 (`#testReplyPath`).
  - `doc.go` lines 423 and 560, the `classify.go:22` field comment, `stencils.go`, `markers_test.go`, the `webster-body-verify-fix.md` leading comment and the `template_test.go` rows at 972 and 1038 are covered by card 3's existing targets.
  - `reset.go:347` calls `StrandLive` inside `refuseReportHeadBatch`, which is on card 3's **Edit** list.
  - `docs/benchmarks/test-redundancy.md` is a measurement report, so it is exempt.
- **F1:** card 10's only remaining "transient" says the errors carry no shed transient mark.
- **F3:** the only text claiming the guard is pure is `goto.go:19`, and card 10 now edits it through `internal/shedengine#GotoGuard`. `shedengine/doc.go:78-80` and `shedverbs/doc.go:16` stay true, so leaving them under Uses is right.
- **F4:** no spec or `docs/overview.md` describes the guard's two arms. `internal/loomcli/doc.go` is the only one, and it is now under **Extends**.
- **Renumber:** all 16 bare card numbers `insert-card` listed were moved to the right new numbers.

Tool calls:
1. grep for the deleted member names across `internal`, `contracts` and `docs`, to rerun F2's site search.
2. grep the plan and `reset.go` around line 347, because the `StrandLive` caller at `reset.go:347` was not in my context.
3. grep card 10, the shed docs and the specs, to rerun the F1, F3 and F4 searches.
4. grep the guard mentions in the `shedverbs` and `shedengine` `doc.go` files, which my context did not show.
5. `sed` on `shedengine/doc.go` lines 76-82, to check that its guard paragraph stays true.
