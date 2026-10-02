# Webster round 1 — fixer report

All five findings are fixed, one commit per fix (F1 split into three per-package commits).
At the final commit, `go build ./...`, `go test ./...`, `go test -tags integration ./...` and `go vet -tags smoke ./...` all pass.

## Fixed

- **F1 (MEDIUM), comment line breaks.** Rewrapped every comment sentence the diff adds or changes to one line per sentence, breaking further only at independent-clause boundaries, per CONSTRAINTS.md's Comment Line-Break Convention.
  Covers `shuttleengine` (`gate.go`, `run.go`, `wait.go`, `gate_list_test.go`), `burlerengine` (`engine.go`, `profile.go`, `engine_test.go`), `shedadapters`, `websterengine`, `shedrecipe` (`config.go`, `entries_gate.go`, the gate-capable entries, their tests and fixtures), the `shedbuild` and `loomrecipe` fixtures, `loomrecipe`'s `gatequiescence_test.go` and `resume_test.go`, and `loomcli`'s `smoke_test.go` and `smoke_gate_test.go`.
  Where a changed sentence shared a line with an unchanged neighbour, the neighbour sentence was also rewrapped so that no fragment was left dangling.
  Commits `91a7bfac5`, `9caa49142`, `f157e240f`.
- **F2 (LOW), miscounted exported types.** The `gate.go` file doc now names all seven exported gate types and drops the count.
  Commit `2724f2bef`.
- **F3 (LOW), hollow profile-sub-map guard.** `GatesRejectedInsideProfileSubMap` (renamed from `GateAttemptsRejectedInsideProfileSubMap`) now puts the row-level `gates` key inside `profile:` and asserts an unrecognized-key error that names `gates`.
  Commit `7e44066c1`.
- **F4 (LOW), unknown name lost element context.** `resolveGateClosure` takes the element index, and its unknown-name error now reads `config key "gates" element <i>: config key "name" must be ...`.
  The new `UnknownNameNamesElement` rejection case asserts that a bad second element is reported as `"gates" element 1`.
  `resolveGateEntry`'s doc now says it wraps config-accessor errors only, which is what it does.
  Commit `655783fe4`.
- **F5 (NIT), stale fake doc.** `loomrecipe/fixture_test.go`'s `RunGated`/`AttachGated` doc now describes the list evaluation.
  `shedrecipe/entries_bouncer_test.go`'s `judgeSeamFakeShuttle` doc had the same stale "invoke the closure exactly once" wording, and it was fixed in the F1 commit `f157e240f`.
  Commit `3f0c926d7`.

## Deferred

None.
