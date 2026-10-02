# Webster round 1 — fixer report

All five findings are fixed, one commit each, none pushed.
The plan's verify block (`go build ./...`, `go test ./...`, the integration-tagged run and the smoke-tagged `go vet` over the named package trees) passes on the fixed HEAD `33a45197f`.

## Fixed

- **F1 (MEDIUM)**, `8b6eba25d`: `RetireLegacy` now saves `board.json` with `legacy_done` still in place, deletes the legacy files and swap locks in the after-save step, and only then drops `legacy_done` and saves again.
  A crash or failed deletion now leaves the surviving legacy files beside an intact `legacy_done`, so a reload folds nothing twice.
  New test `TestRetireLegacyFailedDeletionKeepsLegacyDone` blocks the deletion with a non-empty directory at `tasks.json.swaplock` and asserts that a reopened entry stays reopened.
  It fails on the old code and passes on the fix.
- **F2 (LOW)**, `8ed8266a6`: `task.go`'s file and `Task` doc now say `board.json`, `Store.MergeTasks`'s doc names `boardCriticalSection` in place of the deleted `writeOp`, `Board.Promote`'s doc names `slug`, and both `ReadBoardType` comments (`battencli/wire.go`, `battenshed/deps.go`) say a recipe is corrected.
- **F3 (LOW)**, `e0a3c62d0`: `RenderCompact` measures and pads with `utf8.RuneCountInString`.
  New test `TestRenderCompact_AlignsMultiByteTitles` covers an em-dash title followed by a status.
- **F4 (LOW)**, `33a45197f`: every new or changed comment listed in the review now has one sentence per line, broken further only at an independent-clause boundary.
  This touches `legacy.go`, `board.go`, `store.go`, `render.go`, `cli.go`, `text.go` and the listed test files.
  The garbled `TestValidateDependencyErrors` doc ("…, and (the tier rule has its own test).") now reads as a complete sentence.
  The same commit runs `gofmt` on `render.go` to drop a trailing blank line the diff left at the end of the file.
- **F5 (NIT)**, `4188bc8fc`: `PromoteNote` reports `task not found`, and `TestCLIPromoteNote_NeverANote` now asserts that message.

## Deferred

None.
