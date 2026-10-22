# Card 6 — shedrun-gate-findings-file

**Create:**
- `plan:internal/shedrun#GateFindingsFile` -> `func GateFindingsFile(l *lyxcwd.Location, runID, row string) string`

**Edit:**
- `internal/shedrun#TestPathConstructors`

**Intent:** D4's path: the per-row file that keeps an exhausted gate's findings is declared in `internal/shedrun` alone (`PATTERN-shed-run-directory`), never tracked, under `.lyx` (`PATTERN-durable-vs-ephemeral-state`).

`GateFindingsFile` returns `<ScratchDir(l, runID)>/gate-findings/<row>.md`, with the segment and suffix as package constants beside the other file names in `paths.go`; its doc says one file per row, written by the row's producer when its gate is exhausted and removed by the row's next call whose gates pass.
Neither `lyx loom goto`, `lyx loom resume` nor a pause removes anything under the scratch directory, so the file outlives them.

`TestPathConstructors` gains the row `{"GateFindingsFile", ... GateFindingsFile(l, "self", "Darn-Write") ...}` wanting `filepath.Join(ScratchDir(l, "self"), "gate-findings", "Darn-Write.md")`.

**ImpactSummary:** one new path constructor and one table row in `internal/shedrun`; nothing reads it until card 8.
