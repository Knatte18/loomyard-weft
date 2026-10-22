# Card 22 — batten-resume-in-window

**Create:**
- `plan:internal/battencli#ResumeInWindow` -> `func ResumeInWindow(location *lyxcwd.Location, slug string) (map[string]any, error)`

**Uses:**
- `internal/battencli#battenCLI.runInWindow`

**Intent:**
The in-process resume `lyx adopt` runs for each adopted task (D5).
`battencli.ResumeInWindow` runs `lyx batten run <slug> --window`'s body (`runInWindow`) for that slug and returns its envelope, so the run's own driver lock refuses a second driver.

**ImpactSummary:** battencli gains one exported function; no existing caller changes.

**Commit:** 22: batten: resume a slug's run in its window in-process
