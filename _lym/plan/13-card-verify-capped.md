# Card 13 — card-verify-capped

**Create:**
- `plan:internal/gatecli#runCardLine` -> `func runCardLine(out io.Writer, card planparser.Card, how string) int`

**Edit:**
- `internal/gatecli#cardVerifyCmd` -> `func cardVerifyCmd(seams detachSeams) *cobra.Command`
- `internal/gatecli#runCardVerify` -> `func runCardVerify(ctx context.Context, out, stderr io.Writer, id string, seams detachSeams) int`
- `internal/gatecli#newCommand`
- `plan:internal/gatecli#runDetached`
- `plan:internal/gatecli#runFront`
- `plan:internal/gatecli#detachRun`
- `plan:internal/gatecli#TestGateTest_DetachedRun`
- `internal/gatecli#TestGateCardVerify`
- `//internal/gatecli/doc.go`

**Uses:**
- `plan:internal/gatecli#frontDecision`
- `plan:internal/gatecli#detachKeyDir`
- `plan:internal/gatecli#detachRequest`
- `plan:internal/gatecli#runPhase`
- `plan:internal/gatecli#detachRun.writePhase`
- `internal/cardverify#Runner`
- `internal/cardverify#IsGateCall`
- `internal/proc#KillTree`
- `internal/proc#HoldNamedJob`
- `internal/proc#TreeRecord`
- `internal/gatecli#cardMatches`

**Intent:** D6: `lym gate card-verify` takes the capped form D2 admits, through the same front and runner.

`newCommand` builds `cardVerifyCmd(seams)`, and its RunE calls `runCardVerify(ctx, out, stderr, id, seams)`.
After the plan read and the card match, which refuse as today before any front, a card with a `**Verify:**` line asks `frontDecision(location's worktree, false, seams)`, since the verb has no `--wait`: not capped, it runs the line in-process as today; capped, it calls `runFront` with verb `card-verify`, the worktree as run directory and the matched card's `card.ID()` as the only word, so `4` and `4-<slug>` key one run.
`runCardLine(out, card, how)` holds the verb's closing, shared by the in-process path and the runner: it prints `gate card-verify: card <id> verify <line> <how>` for a failure and returns the line's own code, 1 for a failure without one, and 0 for a pass.

The runner (`runDetached`) runs a `card-verify` request by reading the plan again in `Cwd`, matching the card id, and running its line through a `cardverify.Runner` over the worktree, holding the hub's pool and wait directory inside a hub, with the runner's lease-cancelled context:
its `Phase` writes `phase.json`, the state `slots` with `Line` set while a slot-held line waits, and `running` with `Line` set once the line runs, so a `--wait` gate line, which takes its own slots inside its child, is named running with no `slot_free` answer;
its `Tied` writes the line's `proc.TreeRecord` to the run directory's `line.json`, a file `detachRun` now names.
When the plan read or the card match fails in the runner, as when the plan was edited between the front's read and the runner's, the runner writes the in-process refusal's text, with its way forward, to `stderr` and an unmarked exit `{Code: 1}`, so the front replays the refusal and exits 1, and the next call, finding `delivered`, starts afresh.
On Windows the runner first holds a kill-on-close job, `proc.HoldNamedJob("lym-gate-" + run id)`, which the line's tree inherits, so the runner's death ends the whole tree; on Linux the line's shell gets the parent-death SIGKILL through `verifyrun.RunTied` (card 6).
A job that cannot be held is logged at Warn and the line runs without it; Bound: on Windows such a runner's death then leaves the line's tree to its own end, since `proc.KillTree` kills nothing there.
A cancelled line's process group gets SIGTERM and then SIGKILL, and a `--wait` gate child cancels on that SIGTERM as a terminating signal does, killing its tiers and releasing its slots.

`runFront`, when it supersedes a run whose request verb is `card-verify` and whose runner is not live, kills the group `line.json` records through `proc.KillTree`, which reaches the shell's descendants the parent-death signal misses, a `--wait` gate child among them, before it starts the fresh runner.
Bound: on Linux a dead runner's line runs on until a front supersedes it, or to its own end when no front comes.
Bound: the key is the card's id, so a call after an edit of the card's `**Verify:**` line joins the run of the old line while its lease holds, as card 11's bound on plan edits states.
The card-verify help says an agent-strand call runs the line in a detached runner and answers `done: false` with exit 75 when the cap passes; the doc's Card verify section says the same and names the key.

Tests: `TestGateCardVerify`'s in-process rows run with no strand and stay as they are, and its annotation subtest builds the verb as `cardVerifyCmd(defaultDetachSeams())`.

`TestGateTest_DetachedRun`, tier1, gains rows over a plan written with `plankit` into the prime, each call `card-verify <id>` with a parsing strand and `helperCapEnv`:
- a line `sleep 3; echo done-line; exit 4` against a 1s cap exits 75 with `done: false` and phase `running`; the same command after it ended prints `done-line` and exits 4;
- a call with `4` while the line runs and a call with `4-<slug>` both answer from one run directory, and the line, which appends to a marker file, ran once;
- with both slots held by the test, a call on a line `true` answers 75 `done: false` with `slot_free: false`, phase `slots` with the line, and both holders named; releasing the slots lets the next call replay exit 0;
- a line `sleep 60` that holds the runner's slot is ended when the lease passes with no second call (1s cap, 1s grace): the sleep is gone and the slot free;
- a `--wait` gate line run through a `lym` stub on `PATH`, a script that execs the test binary in the gate helper role over `test --wait ./pkg` with a holding stand-in, is ended the same way: the stand-in is gone and the pool holds no slot afterwards;
- with `helperLaunchEnv=late`, a 4s claim bound and a 6s cap, the test rewrites the plan without the card while the runner's start is delayed: the runner records exit 1 with the refusal naming the plan's ids, and the call replays it and exits 1;
- after the runner of a line `sleep 60 & echo $! > <file>; wait` is SIGKILLed, the next call kills the recorded group, the backgrounded sleep is gone, and a fresh run starts.

**ImpactSummary:** `card-verify`'s builder and run function take the detach seams, with `newCommand` the one caller; the runner and the front gain the card-verify branch, and an in-process card-verify call runs as before.
