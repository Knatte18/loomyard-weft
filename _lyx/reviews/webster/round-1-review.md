---
verdict: APPROVED
---

Review range: `git diff $(git merge-base main HEAD)..HEAD`.
`product.parent` is `main` and no `_lyx/loom/rework/` round exists.
No publish failure is on record.

Card 1: `goPackageDir` is applied in `targetDir` to glyph-derived units only, so member glyphs and `plan:` handles reach it and path refs and file units do not.
`brokenFabricVocabularyRule` uses it.
`PATTERN.md`, the chokepoint background file, `doc.go` and the new rows (`TestCardTargetDirs`, `TestCardGateCommand`, `TestPeakContext`) match the card.
Merging `internal/x` and `internal/x_test` into one entry is pinned by a row.

Cards 2 and 3 apply the same rule: relative spellings only, `.go` arguments skipped, a trailing `/...` removed, and the same `_test` replacement hint.
`runTest` refuses right after `absoluteDir`, before signal handling, hub resolution, slot acquisition and the prebuild.
The in-hub ordering is pinned by the scenario's `refusals` row.
The plan check's `--`, `-C` and `--tags` handling matches the gate's parsing, which splits arguments at `--`.

Card 4: `status-missing` and `status-unreadable: <error>` are told apart through the `state.ErrDecode` check.
A decode error stops at once, and any other read error gets the single shared re-step.
The error text names the status file and the loop log.
`trace_file` and the hook receive the loop log.
`WriteFailedStop` is skipped for a status stop, so the file is left as found.
Each way forward is split by cause as the card says.

Tests: no new top-level test beyond `TestValidate_VerifyPackageMissing`, and the planparser budget row is raised by one.
Each added row covers a behavior no existing test covers.
`go`'s and `lyx gate test`'s package-argument handling has no per-helper test that the public-surface tests already cover.
`lyx gate test` on `planparser`, `gatecli`, `shedverbs`, `shedengine`, `batcher` and `websterengine` passes.
