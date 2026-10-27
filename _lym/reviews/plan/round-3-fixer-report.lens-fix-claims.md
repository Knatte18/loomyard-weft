```yaml
lens: fix-claims
outcome: reported
```

Task: check that each claim the fix added to card 8 holds against the code at HEAD (fix-claims lens).

No findings.

- **F1 text:** `ownerRead` is a `logger.Episode` field of `daemon` (`internal/dobbyengine/loop.go:166`). `beamOwnedElsewhere` logs an unreadable owner through its own episode, with `Fail` on error and `Recover` on success (`loop.go:250-254`). So "a new `stateRead` field of `daemon` beside `ownerRead`, as `beamOwnedElsewhere` logs an unreadable owner" holds. A decode failure comes back as `ErrDecode` and can be told apart with `errors.Is` (`internal/state/state.go:194`, `internal/dobbyengine/state.go:87,103`). So the condition "an error other than a decode failure" can be carried out.
- **F2 row:**
  - `ReadJSONNoMkdir` returns `ErrDecode` for bytes that are not JSON (`state.go:193-194`), and the card reads that as absent, so `keptTidy` becomes nil.
  - With `.git` present, the post-tick test does not exit.
  - The `Sleep` wrapper then removes the worktree.
  - Iteration 2's top-of-iteration check (card line 33) exits before any tick's `UpdateState` could rewrite the record.
  - So "no `StopStrand`" follows.
  - The harness has an injectable `Sleep` and a `stopAt` cancel (`internal/dobbyengine/loop_test.go:72-96`), so the row can be built.
- **`Uses` entry:** `internal/logger#Episode` exists (`internal/logger/episode.go`).

Tool calls:
1. Grepped `internal/dobbyengine/loop_test.go` for the harness's `Sleep` and `stopAt` seams, and grepped `ErrDecode` across `internal/dobbyengine` and `internal/state`. My context lacked the harness shape and how callers tell a decode failure apart.
