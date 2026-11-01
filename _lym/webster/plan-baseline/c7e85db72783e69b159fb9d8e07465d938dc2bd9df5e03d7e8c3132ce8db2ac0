# Card 4 — An in-process batten cancel that tells nothing-left apart

**Create:**
- `plan:internal/battencli#ErrNothingLeft` -> `var ErrNothingLeft = errors.New("nothing of the slug is left to cancel")`
- `plan:internal/battencli#Cancel` -> `func Cancel(ctx context.Context, prime *lymcwd.Location, slug string, remove bool) error`
- `plan:internal/battencli#cancelJudgingEndOnly` -> `func cancelJudgingEndOnly(ctx context.Context, slug string, remove bool, deps cancelDeps) error`

**Edit:**
- `internal/battencli#refuseNothingLeft`
- `internal/battencli#TestCancelRun`
- `internal/battencli#TestCancelPauseEntries`

**Uses:**
- `plan:internal/pauserecord#HoldCaller`
- `internal/battencli#newCancelFake`
- `internal/battencli#cancelFake`
- `internal/battencli#cancelRun`
- `internal/battencli#hubCancelDeps`

**Intent:**
D8's removal runs `lym batten cancel <slug> --remove` as its first step and counts cancel's nothing-left refusal as that step done, while any other refusal fails the step.

`refuseNothingLeft`'s two `nothing of %s is left to cancel` returns keep their text and match `ErrNothingLeft` through `errors.Is`, by an error value whose `Error()` is the text and which wraps or `Is`-matches the sentinel; the task-branch refusal, the origin-unreachable refusal and the probe refusals do not match it.

`Cancel` is the in-process cancel `cobra`'s `cancel` verb runs for a slug addressed from the prime: it builds `hubCancelDeps(prime, slug)` and returns `cancelJudgingEndOnly(ctx, slug, remove, deps)`, so `fronttask`, which runs from the prime's location in the adopt check, ends a run without spawning the binary.
`cancelJudgingEndOnly` runs `cancelRun(ctx, slug, remove, deps)` and returns its error; when `cancelRun` returns no error, it returns nil for a full cancel, and for a report reading `EndOnly` the error `batten cancel of <slug>: origin was unreachable, so the cancel ended the run's processes and stopped; way forward: run <command> again once origin answers`, the command being `cancelCommand(slug, remove)`, `lym batten cancel <slug> --remove` for `fronttask`'s call, which does not match `ErrNothingLeft`.
An end-only cancel leaves the entry's run status and the run record in place, so a caller that counted it done would go on to a removal the run lock refuses; D8 makes a cancel that cannot reach origin fail the step.
The verb itself keeps its end-only report and exit, since its operator reads the report.
`Cancel` skips the verb's own self-address and own-pair refusals, which guard a cancel typed inside the pair; its caller never runs inside a task pair (PATTERN-pair-teardown).
Bound: it adds no path past cancel's own refusals; every refusal `cancelRun` makes still stops it.

`Remove` runs the cancel only when the slug reads a run claim, record or pair, each of which `refuseNothingLeft` counts as left, so the nothing-left match is reached there only when another cancel of the slug finishes between that read and this cancel; the match keeps that race from failing the removal.

D3's hand lift: `lym batten cancel <slug>` lifts every live entry of the slug's own scope through `liftSlugPauses`, whatever its caller, so it lifts a `HoldCaller` entry; `TestCancelPauseEntries` gains the row of an own-scope `HoldCaller` entry, lifted and never refusing.

`TestCancelRun` gains assertions on its nothing-left rows: the error matches `ErrNothingLeft` through `errors.Is` and its text is unchanged; and on its task-branch-only and origin-unreachable rows that it does not.
On its end-only rows, `cancelJudgingEndOnly` over the row's fake returns an error naming origin unreachable and `cancelCommand(slug, remove)` of the row, `--remove` included only on the remove rows, not matching `ErrNothingLeft`, while the entry keeps its run status and the run record stays; on a full-cancel row it returns nil.

**ImpactSummary:** refuseNothingLeft's two nothing-left errors gain a sentinel match with unchanged text; one new exported wrapper over a new cancelRun composition that reads an end-only cancel as an error; no caller changes.

**Commit:** 4: batten: an in-process cancel whose nothing-left refusal is matchable
