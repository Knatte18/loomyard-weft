# Card 13 — card-gate-runner-fixer

**Create:**
- `contracts/stencils/gatefix/gatefix-body-card.md`
- `contracts/stencils#GatefixBodyCard`
- `internal/gatecli#phaseFixing`
- `internal/gatecli#cardFixerKit`
- `internal/gatecli#handsOff`
- `internal/gatecli#newCardFixer`
- `internal/gatecli#newFixerStopper`
- `internal/gatecli#keyFixerStore`
- `internal/gatecli#treeIdentityAt`
- `internal/gatecli#detachRun.runWithFixer`
- `internal/gatecli#detachRun.watchPhase`
- `internal/gatecli#fakeCardFixer`
- `internal/gatecli#TestHandsOff`
- `internal/gatecli#TestRunWithFixer`

**Edit:**
- `internal/gatecli#detachSeams`
- `internal/gatecli#defaultDetachSeams`
- `internal/gatecli#joinState`
- `internal/gatecli#joinable`
- `internal/gatecli#front.readJoinState`
- `internal/gatecli#front.treeIdentity`
- `internal/gatecli#runnerCmd` -> `func runnerCmd(gateTree func() *cobra.Command, seams detachSeams) *cobra.Command`
- `internal/gatecli#runDetached` -> `func runDetached(ctx context.Context, out io.Writer, runDir string, gateTree func() *cobra.Command, seams detachSeams) int`
- `internal/gatecli#newCommand`
- `internal/gatecli#TestGateRunner_Refusals`
- `internal/gatecli#TestJoinable`
- `internal/gatecli/doc.go`

**Extends:**
- `contracts/stencils#entries`
- `contracts/stencils#TestStencils_TopLevelMarkers`

**Uses:**
- `internal/gatecli#detachRun.runRequest`
- `internal/gatecli#detachRun.cancelOnPassedLease`
- `internal/gatecli#detachRun.expireLease`
- `internal/gatecli#originSlug`
- `internal/gatecli#loadGate`
- `internal/gatecli#writeLiveLease`
- `internal/gatecli#writeRequest`
- `internal/gatecli#TestGateTest_DetachedRun`
- `internal/testkit/stencilkit#Seed`

**Intent:**
A red `lym gate test --gate <key>` agent call hands its failure to a cold fixer when the key's `fixer` is set, then answers with the re-run's result (D4).

- `handsOff(request, cfg, slug)` returns the request's `--gate` key and true only when the verb is `test`, the request's role is not in `gatefixer`'s fixer role set (`gatefixer.IsFixerRole`, numbered forms included), the call names a gate key whose `fixer` is non-empty, and `slug` is non-empty; config load allows `fixer` on the `card` key alone (card 5), so only the Webster card gate hands off.
  Only a capped call reaches a runner at all, so an operator's call, a `--wait` call and a call inside an enclosing slot never hand off.
- `newCardFixer(request)` resolves the request's worktree from its `Cwd`, loads the gate config with `loadGate`, reads the slug with `originSlug`, and when `handsOff` holds builds a `cardFixerKit`: a `gatefixer.NewStrandFixer` over a shuttle runner built on the worktree's reed geometry as `pairteardown`'s cancel builds one, registered through `gatefixer.Registrar` under the slug with role `gate-fixer`; the model choice resolved from the key's `fixer`; the gate config's `FixerTimeout()`; the stencils dir and the hub's language skills.
  It returns false, and the call ends as today, outside a hub, with no slug, or for a key or role `handsOff` turns down; a load or resolve error is returned, and the runner logs it and ends the call as today.
- `newFixerStopper(request)` builds the strand stop over the worktree's reed geometry alone, whatever `handsOff` answers, so a live fixer of an earlier run is stoppable even when the key's `fixer` was since emptied or the kit cannot be built.
- `detachSeams` gains `cardFixer func(detachRequest) (cardFixerKit, bool, error)`, `newCardFixer` by default, `fixerStopper func(detachRequest) (func(guid string) error, error)`, `newFixerStopper` by default, and `treeIdentity func(dir string) treeRecord`, `treeIdentityAt` by default; `runnerCmd` and `runDetached` take the seams so the runner reaches them, and `newCommand` passes its seams to `runnerCmd`.
- `keyFixerStore` is a `gatefixer.Store` over `fixer.json` in the run's key directory, the run directory's parent, written through `internal/state`, so the record of a live fixer outlives its run (D8).
- `watchPhase(ctx)` sets up, around a phase outside `runRequest`, the same two cancels `runRequest` sets up around the gate run: `catchTerminatingSignals` and `cancelOnPassedLease`; it returns the phase's context and a closer that ends the watch and answers the abandoned exit, `{128 + signal, abandoned}` after a signal or `{1, abandoned}` with `abandonedLine` on stderr after a passed lease, and false when neither ended the phase.
- `runDetached` runs `runWithFixer` in place of `runRequest`:
  1. Before its gate run, on a live record in the key directory, it writes the phase `fixing` and, under `watchPhase`, waits on it through the kit's `Await` until terminal and marks it terminal.
     When it cannot attach (no kit, or an `Await` error) it stops the strand through `fixerStopper` and marks the record dead `died` only once the stop succeeded; a failed stop leaves the record live and ends the call with exit 1 and a stderr line naming the strand and the way forward to re-run the gate call, whose next run retries the stop, so the gate never runs over a fixer's half-done edits and no second fixer starts beside it.
  2. It runs `runRequest`; a pass, an abandoned exit, a `card-verify` request, or a request `cardFixer` turns down or errors on, is the run's result.
  3. On a red verdict, after `runRequest` returned and released its slots, it writes the phase `fixing`, then the stderr report to `fixer-report.md` in the run directory, and spawns through the kit with the frame and `gatefix-body-card`, filled with the report, the gate call as `lym gate test` plus the request's words, the `--card` ids the words name and the worktree root, the reply at `fixer-reply.md` and no parent name, so the prompt renders the parent directive's no-parent variant (D4).
     A spawn error ends the call with the first red verdict and a stderr line naming the spawn error and that no fixer ran.
     It waits on the fixer in slices of a second under `watchPhase`.
  4. When `watchPhase` ends the phase first, by a passed lease or a terminating signal, it stops the fixer, marks the record dead only once the stop succeeded, and returns the abandoned exit `watchPhase` answers.
  5. Otherwise it marks the record terminal, rewrites `tree.json` with the `treeIdentity` seam over the tree the re-run starts on, runs `runRequest` again, and appends to its stderr a `## Cold fixer reply` section with the reply, or the dead note naming the reason and that any uncommitted change is in the worktree; the run's exit is the re-run's.
- `treeIdentityAt(dir)` is the tree identity `front.treeIdentity` builds, HEAD's tree and the sorted dirty paths, moved out so the runner can rewrite it; `front.treeIdentity` calls it.
- `joinState` gains `fixing`, read by `front.readJoinState` from the run's phase file; `joinable` skips the tree-identity comparison while `fixing` holds, so a caller's repeated call joins whatever tree the fixer is changing, during an attach wait as during its own fixer, and from the re-run's start the rewritten `tree.json` holds the rule again, so a call after the re-run's exit joins and gets its result.
  The phase is written before the spawn and before an attach wait, so no window lets the fixer change the tree while the phase still reads otherwise.
  The join stays keyed by the verb, run directory and words, so only a call with the same key joins.
- `gatefix-body-card.md` sets the card fixer's scope: source in the worktree, reading the card and the uncommitted diff; no commit, since the fork commits the card; re-check with `{{.gate_call}}`; the shared rules and leave-list of card 9's bodies.
  It is embedded and registered with its marker row.
- `doc.go` documents the hand-off, the fixer phase and its join rule.

Bound: one fixer per red call, which Go bounds; the number of red calls per card rests on the fork stencil's self-fix cap.
A fixer's own gate call never hands off, and it is keyed apart by card 12.

Tests, tier0:
- `TestHandsOff` rows: a `card` key with `fixer` and a slug hands off; an empty request role, as an older front writes, hands off; a `gate-fixer`, `gate-fixer-2` and `verify-fixer` role do not; no `--gate` word, a key without `fixer`, an empty slug and a `card-verify` verb do not.
- `TestRunWithFixer` over a scripted gate tree, `fakeCardFixer` (a scripted kit that records the spec it is handed, writes a reply or ends dead, and can fail its spawn, its `Await` or its `Stop`), a recording stopper and a scripted `treeIdentity` seam, in a temp key and run directory holding a live lease from `writeLiveLease` and a request from `writeRequest`, with stencils from `stencilkit.Seed`:
  - a green call spawns none;
  - a red call writes the phase `fixing` before it spawns, spawns one, re-runs, exits with the re-run's code and its stderr carries the reply; the spec handed to the kit carries the gate call, the `--card` ids, the reply path and the parent directive's no-parent variant;
  - a dead fixer's re-run carries the dead note; a spawn error ends with the first red verdict and the spawn line;
  - a turned-down request and a `cardFixer` error spawn none and end with the first verdict;
  - a live record in the key directory is waited on under the `fixing` phase before the gate runs, and no fixer spawns beside it;
  - an unattachable live record with no kit is stopped through `fixerStopper` and marked dead before the gate runs; a failing stop leaves it live and ends the call with exit 1;
  - a lease lapsed with `expireLease` during the fixer phase stops the fixer and exits `{1, abandoned}` with the abandoned line;
  - `tree.json` holds the identity the `treeIdentity` seam answered at the re-run's start.
- `TestJoinable` gains a row: a run in the `fixing` phase joins under changed dirty paths; the existing changed-dirty-paths row pins the run that is not fixing.
- `TestGateRunner_Refusals` passes the seams and keeps its refusals.
- `TestGateTest_DetachedRun`, tier1, stands unchanged: its stand-in calls have no `fixer` key, so they never hand off.

**ImpactSummary:** the detached runner gains a fixer phase reached only for a red `card` call from a non-fixer strand on a pair, or for a live fixer record an earlier runner left; `runnerCmd` and `runDetached` are re-signed with their two callers here; every other detached call runs and joins as today.
