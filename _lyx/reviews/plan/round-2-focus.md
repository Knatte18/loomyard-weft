---
round: 2
focus:
  - "Any card that adds a func-typed field defaulting to a stdlib call when nil to a struct reachable from `shedrecipe.Env` (for example `battenshed.NoticeLog.Now` in `_lyx/plan/08-batten-notice-log.md`): check that the card lists every nested path in `internal/testkit/envkit`'s `nilLegal`, since `envkit.NilSeams` at `internal/testkit/envkit/envkit.go` reports every unlisted nil func field and `TestWire_EverySeamFilled` in `internal/battencli` would fail."
  - "`_lyx/plan/01-render-size-vocabulary.md`: check that its Intent's statements about which files stay untouched agree with its **Edit:** targets, in particular `internal/reedengine/render#placement`, declared in `layout.go`."
  - "`_lyx/plan/07-*.md`, `_lyx/plan/09-batten-overview.md` and the overview's shared decision on minute-resolution times: check that a formatter both panes need has one owner, not a copy per package, and that every card using it lists it under **Uses:**."
  - "`_lyx/plan/09-batten-overview.md` **Tests:** and its `test-budget.yaml` bullet: check that no top-level test covers a single helper of a behaviour another planned test already pins, and that the budget arithmetic matches the remaining test count."
---

Focus is on the areas where round 1 found defects, plus the fixes' new text: any card the fixer changed is new text no reviewer has yet seen.
