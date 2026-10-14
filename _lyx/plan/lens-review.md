# Lens review

Every fork returned a report.

- test-economy, LOW, card 1 `TestRenderBatchGate` row duplicates the new `TestCardGateCommand` subtest through the shared `gateScopeOf`: fixed as proposed; the `TestCardGateCommand` subtest and its target are dropped, and the batch row is kept and noted as pinning both gate forms.
- test-economy, LOW, card 1 `TestCardTargetDirs` handle row repeats the member-glyph row's path through `refUnit`: fixed as proposed; the handle row is dropped, with a note that `refUnit` sends a handle down the glyph branch.
- test-economy, LOW, card 3 pass-through subtest partly repeats the nested-module subtest's `./...`: fixed as proposed; the Intent states the subtest exists for the import path, the `.go` argument and the pattern over an existing base.
- criteria-coverage, MEDIUM, card 2 no row reaches `go build` or `go vet`, or the unread `go run`: fixed as proposed; rows `go build ./nope` and `go vet ./nope` report and `go run ./nope` reports nothing.
- criteria-coverage, MEDIUM, card 3 no row has two missing arguments: fixed as proposed; the row becomes `./nope ./internal/x ./gone`, asserting `./nope, ./gone names no directory`.
- criteria-coverage, MEDIUM, card 4 only the first row asserts the loop log as trace file: fixed as proposed; every stopping row asserts `trace_file` and the hook's trace file equal `LogPath`.
- criteria-coverage, LOW, and row-reachability, LOW (merged), card 4 no row reaches a read error after the re-step is spent: fixed as proposed; a row whose first child fails transiently and whose next read fails stops at once.
- criteria-coverage, LOW, card 4 no row asserts D5's `logger.Warn`: fixed as proposed; each stopping row captures the logger through `internal/testkit/logcapture` and asserts the status path, the rows running non-parallel since the capture is process-global.
- criteria-coverage, LOW, card 2 no row reaches the external-test hint from a created directory: fixed as proposed; a Create card on `internal/made/m.go` with Verify `lyx gate test ./internal/made_test` reports with `name ./internal/made`.
- criteria-coverage, LOW, card 2 overview finding detail left half-spelled: fixed as proposed; the overview detail is spelled out in full.
- criteria-coverage, LOW, card 4 `LoopSpec.ReadStatus` seam unlicensed by the record: fixed as proposed; the Intent states it as an implementation choice serving the acceptance row for a read error that clears on the re-read.
- criteria-coverage, LOW, card 3 the integration subtest is the sole proof the refusal precedes the slot and prebuild: disputed, every fix worse; holding a slot or reaching a prebuild needs a hub, which only `hubforge.CopyHub` gives at the integration tier (`internal/gatecli/test_integration_test.go:1` build tag, `:261` `hubforge.CopyHub`), so a unit-tier copy is impossible under `PATTERN-test-speed`, and the plan's `## verify:` runs that tier.
- forced-edits, LOW, card 4 `internal/shedverbs/doc.go:86` says OnExit runs after a failed stop is written: fixed as proposed; the Intent adds that sentence's edit.
- forced-edits, LOW, card 4 `runLoop`'s doc comment says every interrupted stop on a running run writes it failed: fixed as proposed; the Intent prescribes the comment's update.
- forced-edits, LOW, card 3 sandbox S10 lacks the new refusal: fixed as proposed; `tools/sandbox/SANDBOX-CORE-SUITE.md` joins card 3 with a Watch line for `lyx gate test ./no-such-dir`.
- row-reachability, LOW, card 3 the `./internal/x_test` row needs two substrings from one `wantError`: fixed as proposed; the row asserts the one combined substring.
- row-reachability, LOW, card 2 a `../` argument joining outside the worktree is unbounded: fixed as proposed; the Intent skips it as unplaceable and a `go test ../outside` row reports nothing.
