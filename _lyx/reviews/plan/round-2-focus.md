---
round: 2
focus:
  - "Card 8 (`_lyx/plan/08-batten-notice-log.md`) gives `battenshed.NoticeLog` a nil-defaulting `Now func() time.Time` and adds a `NoticeLog` field to `InnerRunDeps` and `RowNoticeDeps`; settle whether every nested nil func path this creates is registered in `internal/testkit/envkit`'s `nilLegal` or otherwise filled by `battencli`'s wiring, so `envkit.NilSeams` and the seam-completeness tests stay green."
  - "Cards 7 and 9 both render times in whole minutes (`12m`, `1h05m`, `2d03h`) for the checklist and overview panes; settle whether the plan carries that format in one shared symbol or two package-local copies."
  - "Card 9 plans its helper tests for the overview frame; settle whether each is a row of the behavior test that already pins the same rendering or a separate top-level test per symbol, against `PATTERN-test-economy`."
  - "Card 1 states which files it leaves untouched while its Edit targets include `internal/reedengine/render#placement`, declared in `layout.go`; settle whether the Intent and the target list agree."
---

Focus targets the open findings' sites only; it caps no severity and pre-states no verdict.
