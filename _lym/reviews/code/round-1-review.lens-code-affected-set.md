```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change, I looked for everything that calls, reads or asserts it, and for the docs and rules that describe it.

**Per change: set found, covered or not**

- **`PastLand` / `LandOutcomeOf`, `landFinished` removed:**
  - A repo-wide search finds no remaining `landFinished`. The only other reader of the Land-row rule is hibernate's trigger, now on `PastLand`; `hibernateengine/doc.go` and `battenrecipe/doc.go` are updated.
  - Covered.
- **`land_notice` key:**
  - Every non-Go mention of beam.yaml keys lives in `internal/dobbyengine/template.yaml`, which carries the key. `README.md:122` only describes `pull_request`, so no doc is stale. `ConfigSpec`, `LoadConfig`, `Config` and `dobbyengine/doc.go` are updated.
  - Covered.
- **`notify_on_land` board field:**
  - Every place the field has to appear is touched: `Task`, `normalizeTask`, `UpsertPayloadKeys`/`upsertAllowedKeys`, `replayFields`, `scopeChanged`'s comment, `boardengine/doc.go`, the upsert `Use` and `Long` text in `boardcli/cli.go`, and `help_test.go`.
  - The consumers `DobbyTasks`, `TaskView` and `BuildView` carry it.
  - A search of non-test Go for other `PullRequest`-style field switches found none that need the new field.
  - Covered.
- **`RecordLanding` seam:**
  - Callers: `landingshed/deps.go`, `finalize.go` (both merge paths) and `loomcli/landingdeps.go`. `envkit.nilLegal` carries `Landing.RecordLanding`. `landingshed/doc.go` and `shedrun/doc.go` are updated.
  - Covered.
- **Landing-commit file:**
  - It sits under `ScratchDir`, so it follows PATTERN-durable-vs-ephemeral-state and PATTERN-shed-driver. The write side (`loomcli`) and the read side (`battencli.DobbyRuns`) both use `shedrun.LandedCommitFile`.
  - `battencli/cleanup.go`'s `emptyScratch` removes the file only after the run's records are removed, which is after the run is unlisted. This is not a defect.
  - Covered.
- **`Deps.ClearNotifyOnLand`, `RunView` and `TaskView` fields, `State.LandNotices`:**
  - The only production `dobbyengine.Deps` constructor is `beamcli/dobbydeps.go`, and it wires `ClearNotifyOnLand`. `Tasks` was already wired.
  - Other `Deps{}` literals are in tests.
  - The `nil` path is documented on the field.
  - Covered.
- **Overview header and `notify_on_land` marks:**
  - `DobbyLines`, `battencli/cli.go`, the `overview.go` help, `beamcli/cli.go` status help and `dobbyengine/doc.go` are updated.
  - Not verified (tool budget spent): `overviewdobby_test.go:111,114` still expect `dobby prime: alive, max_runs 3` with no `land_notice`. They are probably a path outside `DobbyLines`' header, but I did not confirm it.
- **PATTERN.md / `pattern/`:**
  - The diff touches neither.
  - The decision record's Constraints declare no PATTERN changes, so there is nothing to land.
  - The commits carry no `Pattern-Departure:` trailer, and I saw no pattern constraint they would need one for.

**Findings**

1. **LOW / scope** — `internal/boardcli/cli.go` (the upsert `Long` text, changed lines for `brief`, `isolated`, `status` and `labels`). To fit the 2000-byte cap, the diff rewrote four existing field descriptions that the board entry does not ask to change. `summary.md` discloses this. It drops information, for example "shown in board listings" and the `e.g. "active"` example. The alternative was to shorten only the new line.
2. **NIT / consistency** — `internal/battencli/overview.go` (the `overviewCmd` `Long` text) and `internal/beamcli/cli.go` (the `statusCmd` `Long` text). Rewording the help left a very long line ("...on; a run or task that asks for a landing notice shows notify_on_land. Below it comes one" in `overview.go`, and a long run-on line in `beamcli/cli.go`) next to the paragraphs that were hand-wrapped. Cosmetic only.

No other member of the affected set is left wrong.

**Tool calls (6)**
1. Searched for `landFinished`, `alive, max_runs` and `task_headroom`/`board_tidy_every` repo-wide, to find leftover references and docs that list beam.yaml keys. My context had only the diff.
2. Searched `*.md`, `*.yaml` and similar for `pull_request`, to find docs that list beam keys or board fields.
3. Searched non-test Go for `PullRequest`, to find the other places that switch over a per-entry board field.
4. Searched for `RecordLanding`/`MarkTaskDone` and `Deps{` constructors, to find the seam's callers and the `dobbyengine.Deps` constructors.
5. Read PATTERN.md and the durable-vs-ephemeral pattern file, and searched for scratch cleanup, to check whether the landed-commit file survives until the dobby reads it.
6. Read `battencli/cleanup.go` (`removeRunRecords`, `emptyScratch`) and `shedrun/paths.go` (`ScratchDir`), to confirm when the scratch is emptied.
