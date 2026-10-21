# Card 11 — card-verify-runner

**Create:**
- `internal/cardverify/doc.go`
- `internal/cardverify/testmain_test.go`
- `plan:internal/cardverify#Runner` -> `type Runner struct`
- `plan:internal/cardverify#DefaultTimeout` -> `const DefaultTimeout = 10 * time.Minute`
- `plan:internal/cardverify#Runner.RunLine` -> `func (r Runner) RunLine(line string, out io.Writer) string`
- `plan:internal/cardverify#Runner.RunCards` -> `func (r Runner) RunCards(cards []planparser.Card, out io.Writer, stopAtFirst bool) []string`
- `plan:internal/cardverify#IsGateCall` -> `func IsGateCall(line string) bool`
- `plan:internal/cardverify#TestRunner` -> `func TestRunner(t *testing.T)`
- `plan:internal/cardverify#TestIsGateCall` -> `func TestIsGateCall(t *testing.T)`
- `plan:internal/gateslot#HolderEnv` -> `func HolderEnv(base []string, lease *Lease, tier0Parallel int) []string`
- `plan:internal/gatecli#cardVerifyCmd` -> `func cardVerifyCmd() *cobra.Command`
- `plan:internal/gatecli#TestGateCardVerify` -> `func TestGateCardVerify(t *testing.T)`

**Edit:**
- `internal/websterengine#rerunCardVerifies`
- `internal/websterengine#RecordBatch`
- `internal/websterengine#RecordDeps`
- `internal/gatecli#newCommand`
- `internal/gateslot#Limits`
- `internal/gateslot#Config.Limits`
- `internal/gateslot#TestLeaseEnv`
- `internal/gateslot#TestTemplateConfigAndLimits`
- `internal/gateslot/doc.go`
- `cmd/lyx#verifyRunAllowed`
- `docs/overview.md`

**Delete:**
- `internal/websterengine#DefaultCardVerifyTimeout`
- `internal/websterengine#runCardVerify`
- `internal/websterengine#acquireCardVerifySlot`
- `internal/websterengine#TestRerunCardVerifies_AllPass`
- `internal/websterengine#TestRerunCardVerifies_NonZeroExit`
- `internal/websterengine#TestRerunCardVerifies_Timeout`
- `internal/websterengine#TestRerunCardVerifies_MissingDir`
- `internal/websterengine#TestRerunCardVerifies_Slotted`
- `internal/websterengine#verifyCard`

**Intent:**
One runner for a card's `**Verify:**` lines, shared by Go's re-run at record-batch and the fork's new `lyx gate card-verify <card id>` (D8, D13).

`internal/cardverify` is a new package holding the runner the webster re-run used, moved out of `internal/websterengine` so `internal/gatecli` can import it without the engine.
`Runner` holds the worktree, the told gate-slot pool (nil unslotted), the wait directory and the per-line timeout (zero meaning `DefaultTimeout`).
`RunLine(line, out)` runs one line through `verifyrun.Run` in the worktree and returns how it failed, empty on a pass, with output to `out`:
- a line `IsGateCall` accepts, one `lyx gate test` call holding `--wait` and no shell control operator (`;`, `&`, `|`, `&&`, `||`, a newline, a backtick, `$(`, `<` or `>`), runs unslotted and outside the timeout, which the gate's own pre-tier ceiling and tier timeouts replace (card 8);
- any other line, a gate call lacking `--wait` included, takes one slot of the pool, writing the wait record `card verify <card>` while it waits and waiting without a cap, and runs under the timeout counted from the moment the slot is held, its environment from `gateslot.HolderEnv`.
Both strip `PrebuiltLyxEnv` as today.
`RunCards(cards, out, stopAtFirst)` runs each card's `**Verify:**` value, which is the one line after its label, in card order, returning one failure line per failing card in today's `card NN-<slug> verify <command> <how>` shape; with `stopAtFirst` it stops at the first failing card.

`gateslot.HolderEnv(base, lease, tier0Parallel)` is the environment of a non-tier slot holder (D13): `base` with GOFLAGS gaining `-trimpath`, plus `-p=<tier0Parallel>` when it is above 0 and no `-p` when it is 0, and the lease's inheritance variable when `lease` is non-nil.
`Limits` gains `Tier0Parallel`, which `Config.Limits` fills with `tier0`'s effective parallel, machine override included, and 0 when the hub declares no `tier0`, so a holder reads it from the pool's `Limits` at its acquire and needs no config reader of its own.
A gate call inside such a line inherits the slot (card 3), so the slot-held line never takes a second slot.

`rerunCardVerifies` keeps its record-batch contract and failure lines and delegates to a `cardverify.Runner` with output discarded; `RecordDeps` carries what the runner needs, its `VerifyTimeout` kept.
`lyx gate card-verify <card id>` reads the plan through `planparser`, refuses an unknown id naming the plan's ids, and refuses a worktree with no plan naming the way forward to run it from the task worktree whose `_lyx/plan` holds the card; it runs the card's line through the same runner with output passed through to the terminal, exits with that line's exit code, and passes a card with no `**Verify:**` with a message saying so; outside every hub it runs unslotted.
It sits under `lyx gate`, since the fork-mode hook refuses every `lyx webster` command from a fork, with the role audience annotation (`clihelp.AudienceRole`), since the implementer fork's prompt names it.
Bound: the verb runs only lines the plan's card already holds, which the plan gate checked (card 18); it adds no slot rule, since each line is either a gate call or one held slot.

`cmd/lyx`'s `verifyRunAllowed` moves its entry from `internal/websterengine/cardverify.go` to the new runner file with the same reason.
`doc.go` of the new package documents the runner; `docs/overview.md`'s module table gains the package, and the gate row names `card-verify`; `gateslot`'s doc names `HolderEnv`.

Tests: the new `TestRunner` sits in an `integration`-tagged file, since it spawns shells, and the package's new `testmain_test.go` runs it under `gitkit.HermeticGitEnv()` and `tmuxkit.Main` (`PATTERN-test-isolation`).
It covers, against real shell lines and a temp pool: a passing and a non-zero line, a missing worktree, a timed-out slot-held line, a slot-held line whose GOFLAGS carry the pool's `Limits().Tier0Parallel` through `HolderEnv`, `stopAtFirst` stopping at the first failing card, and the record-batch failure line shape; it replaces the five webster `TestRerunCardVerifies_*` tests, which move here with their helper.
One subtest puts a stub `lyx` first on `PATH` with `t.Setenv`, so it is not parallel and names that process-global state: the stub sleeps past the runner's short timeout and exits 0, and the gate-call line passes with no slot held, so no runner timeout bounds a gate call; the gate's own `--wait` row past the keep-cache-warm cap (card 8) covers the busy-hub half of the criterion.
`TestIsGateCall` is untagged and covers the accepted call and a call without `--wait`, with `&&`, with a pipe and with `$(`.
The new `TestGateCardVerify` sits in gatecli's `integration`-tagged file, since its lines spawn a shell: an unknown id and a missing plan refused with their ways forward, a card with no `**Verify:**` passing, a line's output reaching the verb's output with its exit code, a run in a standalone worktree outside every hub taking no slot, and the verb's audience annotation being `clihelp.AudienceRole`.
`TestLeaseEnv` gains the `HolderEnv` rows, the one place the flag rule is pinned: `-trimpath` and `-p=<n>` for a tier0 parallel above 0, `-trimpath` alone for 0, and the inheritance variable for a non-nil lease.
`TestTemplateConfigAndLimits` asserts `Tier0Parallel` for the template (0) and for a config declaring `tier0` with a machine override, the one place that mapping is pinned.

**ImpactSummary:** the card re-run moves into a new package with its tests; record-batch's verdict and failure lines are unchanged, and the gate CLI gains one verb.
