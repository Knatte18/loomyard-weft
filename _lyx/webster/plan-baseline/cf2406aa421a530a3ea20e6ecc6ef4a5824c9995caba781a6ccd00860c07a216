# Card 19 — cleanup-scratch-segment

**Create:**
- `internal/cleanupcli/doc.go`
- `internal/cleanupcli/scratch.go`
- `plan:internal/cleanupcli#scratchDirName` -> `const scratchDirName = ".scratch"`
- `plan:internal/cleanupcli#scratchMaxAge` -> `const scratchMaxAge = 14 * 24 * time.Hour`
- `plan:internal/cleanupcli#scratchSegment` -> `func scratchSegment(dir string, readBoard func() ([]boardengine.Task, error), now time.Time, apply bool) ([]string, error)`
- `internal/cleanupcli/scratch_test.go`
- `internal/cleanupcli/testmain_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/fsx#NewestModTime`
- `internal/boardengine#Task`
- `internal/boardengine#Board.ListTasksFull`

**Intent:**
Start the `cleanup` module, `internal/cleanupcli`, with the segment it owns itself: the prime's `.scratch/`, which no other module owns (D11).
`scratchSegment(dir, readBoard, now, apply)` takes the `.scratch` directory, the board read and the clock as arguments, so a test drives it with no hub.
It reads the board first: a `readBoard` error fails the segment with that error and nothing removed.
A missing `dir` is an empty segment.
A direct child of `dir` is listed when `fsx.NewestModTime` of it is older than `now - scratchMaxAge` and no board entry whose status is not done cites it, where citing is the child's `.scratch/<name>` path appearing in the entry's title, brief or body.
A link directly under `dir` is judged by the same rule on its own modification time and removed as a link, never followed; `fsx.NewestModTime` already answers a link's own time.
With `apply` true each listed child is removed (`os.Remove` for a link or a file, `os.RemoveAll` for a directory); with it false nothing is, and the list is the same.
Bound: only direct children of the prime's `.scratch/`, and only with the board read.
`scratchMaxAge` matches logger's default retention age.

`scratch_test.go`, untagged and parallel over `t.TempDir()` with a fixed `now` and modification times set by `os.Chtimes`, holds one table over the acceptance rows: an old child removed; a child with one new file deep inside kept; an old child cited by a not-done entry kept and one cited only by a done entry removed; an old link removed while its target survives; a missing `.scratch/` an empty segment; a board read error failing with nothing removed; and a dry run listing what the apply row removes while removing nothing.
`testmain_test.go` is untagged and calls `gitkit.HermeticGitEnv()` and then `os.Exit(tmuxkit.Main(m))`, since card 20 adds a hub-backed integration test to the package.
`doc.go` describes the module and this segment; card 20 adds the verb.
`cmd/lyx/testdata/test-budget.yaml` gains an `internal/cleanupcli` row equal to the package's top-level test count.

**ImpactSummary:** A new package with one unexported segment function and its budget row; nothing calls it yet.
