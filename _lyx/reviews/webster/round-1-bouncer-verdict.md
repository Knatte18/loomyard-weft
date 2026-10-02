---
verdict: APPROVED
rationale: "All five round-1 findings are confirmed fixed on HEAD 33a45197f (F1 RetireLegacy keeps legacy_done until deletion succeeds, with a regression test; F2 stale docs refreshed; F3 rune-based padding with a multi-byte test; F4 comments rejoined; F5 task not found), and the plan's full verify block (build, test, integration, smoke vet) passes."
---

Review range: `product.parent` is `main` (`_lyx/shed/bolt-v2/status.json`), merge base `9ae5f9405`.
The diff covers the ten card commits `63e8b1337..4594739bc` and the five fixer commits `8b6eba25d..33a45197f`.

The round-1 report was itself `APPROVED`, and it carried one MEDIUM, three LOW and one NIT finding.
It has no `## Focus departures` section, so there is no departure to ratify or reject.
Its five settled focus directives match the round-1 focus file, and I accept its conclusions on the legacy compatibility window and tier-rule symmetry.

## Fix verification

- **F1 (MEDIUM), `8b6eba25d`.**
  `RetireLegacy`'s mutate now only captures the store.
  `boardCriticalSection` saves `board.json` with `legacy_done` intact, the after-save step deletes the legacy files and swap locks, and only then does it set `legacyDone = nil` and call `Save` again.
  `LegacyDone` is `omitempty`, so the second save drops the field.
  A failed deletion returns before the second save, which leaves `legacy_done` beside the surviving file.
  `TestRetireLegacyFailedDeletionKeepsLegacyDone` blocks the deletion with a non-empty directory at `tasks.json.swaplock` and asserts that a reopened entry stays reopened.
  The doc comment now describes the real ordering.
- **F2 (LOW), `8ed8266a6`.**
  `task.go` names `board.json`, `MergeTasks` names `boardCriticalSection`, `Board.Promote` names `slug`, and both `ReadBoardType` comments say "a recipe corrected".
- **F3 (LOW), `e0a3c62d0`.**
  Widths and `pad` now use `utf8.RuneCountInString`.
  `TestRenderCompact_AlignsMultiByteTitles` covers an em-dash title.
- **F4 (LOW), `33a45197f`.**
  The listed comments now put one sentence per line and break only at clause boundaries.
  A scan of every added comment line in the whole diff finds what remains is either legacy text moved unchanged (the `storeVerbs` verb comments in `cli.go`) or a one-word substitution inside a legacy fixed-wrapped comment.
  The substitutions are `seedBoardTask`, the `boardCriticalSection` guard comment, the `concurrency_test.go` header and `TestLoadCorruptBoardJSON`.
  CONSTRAINTS.md says untouched lines are never rewrapped, so a word swap inside a legacy wrap keeps that wrap and is not a finding.
- **F5 (NIT), `4188bc8fc`.**
  `PromoteNote` reports `task not found`, and `TestCLIPromoteNote_NeverANote` asserts that message.

## Verify block

I ran the plan's verify block on HEAD `33a45197f` myself, and every command exited 0:
`go build ./...`, `go test ./...`, the integration-tagged run over the six named package trees, and `go vet -tags smoke` over the same trees.

No new finding came up while checking the fixes.
