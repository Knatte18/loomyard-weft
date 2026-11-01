```yaml
lens: fix-claims
outcome: reported
```

Fix-claims lens: checking that each sentence the fix added or rewrote holds against the code at HEAD.

**LOW: one row asserts `--remove` for every end-only row of `TestCancelRun`**
- **Changed text:** `_lym/plan/04-batten-cancel-export.md`, the new `TestCancelRun` line: "On its end-only row, `endOnlyError` over the returned report is an error naming origin unreachable and `lym batten cancel <slug> --remove`".
- **Summary:** the end-only rows run with `remove` both false and true, and `endOnlyError` names `cancelCommand(slug, remove)`. The `--remove` assertion therefore holds only for the remove=true rows.
- **Evidence:**
  - `internal/battencli/cancel_test.go:784-791` builds `%s_Remove%v_EndsOnly` rows for `remove` in `[]bool{false, true}`.
  - `internal/battencli/cancel.go:181-186`: `cancelCommand` adds `--remove` only when `remove` is set.
  - The card's own Intent says the command is `cancelCommand(slug, remove)`.

Every other added or rewritten claim holds:
- **Card 4, end-only path:** `cancelRun` returns the end-only report with a nil error before `clearRunPhase` (`cancel.go:200-203`). On the end-only rows, `RemoveRecords` and `ClearClaim` are not called (`cancel_test.go:797`), so the record and the run status stay.
- **Card 4, verb exit:** the verb keeps its end-only report and exits through `output.Ok` (`cancelcmd.go:121-125`).
- **Card 6, Settle bound:** dobby reads its beam's owner at start and on each probe tick, and exits when another machine owns the beam (`dobbyengine/doc.go:39`).
- **Card 5:**
  - `tierNotice(tier, logPath)` (`check.go:275`) and `suiteMessage(suite suiteStart)` (`adopt.go:347`) match the re-sign arrows.
  - `checkMarkOf(record CheckRecord, tierStart time.Time)` (`check.go:241`) can read the subject from the record.
  - The gateslot table row is at `gateslot/doc.go:108`.
- **Card 11:**
  - `ownerWriteError` has two errors that say nothing was resumed (`adopt.go:562-567`).
  - The lost-owner and read-back errors say "nothing was resumed" (`adopt.go:306,314`).
  - `heldBeamWayForward` says the parent's adopt "resumes the landing here" (`adopt.go:672`).
  - `beamcli/doc.go:31` says "only this verb, a hand pass or `lym adopt` starts or resumes its run".
- **Card 12:**
  - `ownerHaltReason`'s `runOwner == ""` case says "which resumes the halted run" (`finalize_owner.go:117`).
  - `battenshed/doc.go:52` says "`lym adopt` and `lym beam land` start a run's first pass detached".
- **Card 13:**
  - `Vocabulary.Known` and `LoadConfig` exist (`boardengine/config.go:114,138`).
  - The template's `labels: {}` holds no `gate` label (`template.yaml:6`).
  - `validateLabels` refuses a label it does not know (`labels.go:18-19`).

Tool calls:
1. `cancel.go:180-192` plus a grep for `EndOnly` and `cancelRun(` across battencli: my context lacked `cancelCommand`'s body, the verb's call site and the test rows for end-only reports.
2. The `cancel_test.go` test function list, `cancelcmd.go:115-135` and `cancel_test.go:780-805`: my context lacked which test holds the end-only rows, their remove variants and the verb's exit on an end-only report.
