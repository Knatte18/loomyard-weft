# Card 6 — Judge a run's status in batten cleanup tolerantly

**Edit:**
- `internal/battencli#judgeStatus`
- `internal/battencli#TestCleanup`

**Intent:**
D4: `judgeStatus`, batten cleanup's read-only judge of a status file's bytes from disk or from the records HEAD, stops calling `DisallowUnknownFields` and decodes with `json.Unmarshal`, so a done run whose status another binary wrote with a key this one does not know is a candidate rather than skipped as undecodable.
A malformed status is still skipped with its decode error, which the existing "undecodable status" row pins.
The `routed_to` key is a history-entry field, unknown at the top level, so it stands for a key a newer binary added.

`TestCleanup` gains one row, "done status with an unknown key is removed, committed and its scratch emptied": a fixture whose `rawStatus` is `{"current_producer":"Worktree-Teardown","state":"done","history":[],"routed_to":"x"}` with `scratchAge: time.Hour`, wanting the slug removed, emptied and committed as the first row does.
Under the strict decode it would land in `Skipped` with a decode reason, so the row fails if the strictness returns.

**ImpactSummary:** one unexported judge in `internal/battencli`, reached only by `lyx batten cleanup`, and one test table row.

**Commit:** 6: judge cleanup statuses tolerantly
