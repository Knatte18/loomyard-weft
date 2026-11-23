```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the fix against the sets its findings name (callers of the changed commit path, notice claims, `judged` uses).

No BLOCKING or MEDIUM findings. One LOW:

- **LOW — `internal/fabricengine/doc.go:110-112`.**
  - The package doc describes `CommitWithTrailers` as committing "as `Commit` does with no tags", with the trailers after `Warp-SHA`.
  - F1's fix changed that contract: on a born warp the weft commit now always lands, empty when the files are unchanged.
  - The fixer updated the doc comment in `commit.go` and `commit weftLocked` but left this package-level twin as it was.
  - The sentence is stale now, because `Commit` with no tags still no-ops on unchanged files.

Sets I checked that hold:

- **F1, callers of the changed behaviour.**
  - The only non-test caller of `CommitWithTrailers` is `webstercli/sync.go:38`, which is the reset path and wants the empty commit.
  - The drift seams (`driftcli/wiring.go:209`, `loomcli/driftwiring.go:190`) use `CommitAnchoredPathsWithTrailers`, a separate function that `forceEmptyCommit` in `commitWeftLocked` does not reach.
  - `commitWeftLocked` has one non-test caller, `commit.go:256`, so no other writer changes behaviour.
- **F2, claims that batten holds its notice.**
  - No remaining text says batten holds the notice for a judged stop.
  - `battenshed/notice.go:337` and `notice_test.go:645` describe the watched marker, where batten's notice waits for the relay and goes when the driver ends. That matches the corrected text.
- **F3, uses of `judged`.**
  - The uses are `drive.go:252` (reset), `:316` (set with the key) and `:1004` (read with the key check).
  - `drivehandback.go:67` reads the already-keyed `h.Judged`. No use was missed.

Tool calls:
1. Grep for `CommitWithTrailers` and `commitWeftLocked(` callers: F1's set, which also showed the doc.go line.
2. Grep for the "holds its own notice" and "batten holds" claims: F2's set.
3. Grep for `.judged`, `judgedKey` and `.Judged`: F3's set.
4. Read the drift wiring seams and `fabricengine/doc.go` lines 106-114: to confirm the drift callers use a different function, and to see the doc.go wording. My context lacked both.
