# Card 6 — one arbiter eligibility function; awaiting narrows to hard-cap and circling

**Create:**
- `internal/arbiterstop/doc.go`
- `plan:internal/arbiterstop#HandBackWayForward` -> `const HandBackWayForward = "way forward: report the stop to the run's parent in the stop report"`
- `plan:internal/arbiterstop#Stop` -> `type Stop struct`
- `plan:internal/arbiterstop#Eligibility` -> `func Eligibility(stop Stop) error`
- `plan:internal/arbiterstop#Readers` -> `type Readers struct`
- `plan:internal/arbiterstop#Check` -> `func Check(status shedengine.Status, kind, transient string, readers Readers) (briefPath string, err error)`
- `plan:internal/arbiterstop#EscalationEligible` -> `func EscalationEligible(producer string, readers Readers) bool`
- `plan:internal/arbiterstop#NewReaders` -> `func NewReaders(location *lyxcwd.Location) Readers`
- `internal/arbiterstop/arbiterstop_test.go`
- `plan:internal/arbiterstop#TestEscalationEligible` -> `func TestEscalationEligible(t *testing.T)`
- `internal/arbiterstop/testmain_test.go`
- `plan:internal/arbiterstop#TestMain` -> `func TestMain(m *testing.M)`

**Edit:**
- `internal/loomcli#arbiterHandBackWayForward`
- `internal/loomcli#arbiterDeps`
- `internal/loomcli#arbiterCall.checkEligible`
- `internal/loomcli#loomCLI.arbiterDepsFor`
- `internal/loomcli#loomCLI.goDriverDepsFor`
- `internal/loomcli#goDriverDeps`
- `internal/loomcli#driveFacts`
- `internal/loomcli#arbiterFixture`
- `internal/loomcli#newArbiterFixture`
- `internal/loomcli#arbiterFixture.deps`
- `internal/loomcli#TestArbiterVerb_SpawnPath`
- `internal/loomcli/doc.go`
- `contracts/stencils/loom/loom-template-arbiter.md`
- `contracts/stencils#wordingClaims`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `internal/loomcli#arbiterStop`
- `internal/loomcli#arbiterEligibility`

**Uses:**
- `plan:internal/shedadapters#PendingEscalationCause`
- `internal/loomrecipe#BouncerRunSubdir`
- `internal/battenshed#RecordMatchesHalt`
- `internal/loomcli#decideDrive`
- `internal/loomcli#TestDecideDrive`

**Intent:**
D5, with D6 unchanged: one Go function answers the arbiter verb's whole `awaiting` and `blocked` eligibility for the verb, the Go driver's `EscalationPending` fact and, from card 8, batten's hold, so the three cannot disagree.
It lives in a new package, `internal/arbiterstop`, because batten must call it and `internal/battencli` cannot import `internal/loomcli`: that would link tree-sitter into battencli's test binary, which `cmd/lyx`'s tree-sitter link guard refuses.
The package's production file `internal/arbiterstop/arbiterstop.go` imports `loomrecipe`, `loomshed`, `loomengine`, `shedadapters`, `shedverbs`, `shedengine`, `battenshed`, `verifytree`, `fabricengine` and `lyxcwd`, none of which links tree-sitter or imports `arbiterstop` back.

`internal/arbiterstop`:

- `HandBackWayForward` is `loomcli`'s `arbiterHandBackWayForward` text moved verbatim, ``way forward: report the stop to the run's parent in the stop report, and `lyx loom status <run-id>` shows the stop``; the declaration head above abbreviates it.
- `Stop` is `loomcli`'s `arbiterStop` moved, with `EscalationPending bool` joined by `EscalationCause shedadapters.EscalationCause` and `EscalationErr error`, the error reading the escalation record returned.
- `Eligibility` is `arbiterEligibility` moved, every refusal still `arbiter: stop not eligible: <reason>; <HandBackWayForward>`.
  Its `awaiting` branch now decides in this order: an `EscalationErr` refuses with `the review escalation at "<producer>" is unreadable: <err>`; no pending escalation refuses as today; a `hard-cap` or `circling` cause is admitted; any other cause refuses with `the review escalation at "<producer>" has cause "<cause>", which goes to the run's parent`, so a `budget` escalation from any source goes to the parent.
  The `blocked`, error-kind, interrupted and other-state branches are unchanged, which keeps D6.
- `Readers` holds `PendingEscalation func(producer string) (cause shedadapters.EscalationCause, briefPath string, pending bool, err error)`, `VerifyRecord func() (battenshed.ChildPublishFailure, bool, error)`, `Head func() (string, error)` and `InterruptPolicy func(producer string) string`, the four reads `checkEligible` makes today.
- `Check` is `checkEligible`'s body over `readers`: it builds the `Stop` from the status, the envelope's kind and transient class and the interrupt policy, reads the escalation at an `awaiting` stop into the three escalation fields and never returns its error itself, reads the verify record and HEAD at a `blocked` Merge-In or Publish stop exactly as today, refusing a read failure of either with `arbiter: the verify failure record: <err>; <HandBackWayForward>` or `arbiter: the pair's HEAD: <err>; <HandBackWayForward>`, and returns `Eligibility`'s answer, with the brief path only for an admitted `awaiting` stop.
  So an unreadable escalation record or an unknown cause is a not-eligible refusal, never an error the caller must handle differently.
- `EscalationEligible(producer, readers)` reports whether `Check` admits an `awaiting` stop at `producer`; it is the Go driver's `EscalationPending` fact.
- `NewReaders(location)` builds the real readers of the task worktree at `location`: the escalation through `loomrecipe.BouncerRunSubdir` and `shedadapters.PendingEscalationCause` under `loomengine.LoomReviewsDir(location)`, reporting not pending at a row that is not a Bouncer; the verify record through `verifytree.ReadPublishFailure` over `verifytree.NewPaths(location.WorktreePath(), verifytree.Dir(location.AnchorPath()))`, mapped onto `battenshed.ChildPublishFailure`; HEAD through `fabricengine.Open(location)` and `HeadSHA`; and `loomshed.InterruptPolicyFor`.
- `doc.go` states the package's job, that the verb, the Go driver and batten's hold share it, and why it is not in `loomcli`.

`internal/loomcli`:

- `arbiterHandBackWayForward` becomes `arbiterstop.HandBackWayForward`; `arbiter.go` and `tackler.go` keep using the local name.
- `arbiterDeps` replaces its `PendingEscalation`, `VerifyRecord`, `Head` and `InterruptPolicy` fields with `Stop arbiterstop.Readers`; `arbiterCall.checkEligible` returns `arbiterstop.Check(c.status, c.stop.Kind, c.stop.Transient, c.deps.Stop)`; `arbiterStop` and `arbiterEligibility` are deleted.
- `loomCLI.arbiterDepsFor` sets `Stop: arbiterstop.NewReaders(location)` in place of the four closures and keeps its `bouncerRunDir` closure for `PendingCirclingDecision`.
- `loomCLI.goDriverDepsFor`'s `EscalationPending` closure returns `arbiterstop.EscalationEligible(producer, arbiterDeps.Stop), nil`, so at a `budget` escalation or an unreadable record the Go driver's fact is false and `decideDrive` takes its existing "awaiting stop with a parent notice: relay it and hand back" row, as the LLM driver does on the verb's refusal (D8).
  `goDriverDeps.EscalationPending`'s and `driveFacts.EscalationPending`'s doc comments say the fact is the arbiter verb's eligibility of the awaiting stop, not mere pendency.
- `doc.go`'s eligibility sentence says an `awaiting` stop is eligible only with a pending escalation of cause `hard-cap` or `circling`, that the rule lives in `internal/arbiterstop`, and its Go driver sentence says the driver asks the arbiter at such a stop only.

`contracts/stencils/loom/loom-template-arbiter.md` drops its `budget` rule line; `wordingClaims`' arbiter entry drops "`budget`" from its D7 `wantAll` and adds a `wantNone` for "`budget`" with the reason that a budget escalation goes to the parent and never reaches the arbiter.
`contracts/specs/refusal-spec.md` gains two loom arbiter rows beside "awaiting without an escalation": "stop not eligible, awaiting at a budget escalation" and "stop not eligible, escalation record unreadable", each with the hand-back way forward.

Tests:
`TestEscalationEligible`, unit tier, is a table over fake `Readers` asserting only `EscalationEligible`'s answer, the Go driver's fact: a pending `hard-cap` and a pending `circling` escalation are eligible; a pending `budget` escalation, one whose read returns an error wrapping `shedadapters.ErrEscalationMalformed`, one whose read returns a plain I/O error, a Bouncer with nothing pending and a non-Bouncer row are not.
The refusal texts and the brief path are the verb rows' to pin.
`arbiterFixture` gains `escalationCause` and `escalationErr`, `newArbiterFixture` sets `escalationCause` to `circling` so every existing awaiting row keeps its meaning, and `arbiterFixture.deps` fills `Stop` from the fixture.
`TestArbiterVerb_SpawnPath` gains three rows at Plan-Bouncer awaiting with a pending escalation: cause `hard-cap`, which spawns with its brief; cause `budget`, refused with `cause "budget"` and spawning nothing; and an `escalationErr` wrapping `ErrEscalationMalformed`, refused with `unreadable` and spawning nothing.
`cmd/lyx/testdata/test-budget.yaml` gains the row `internal/arbiterstop: 1`.

**ImpactSummary:** the arbiter eligibility rule moves out of loomcli into a package batten can import, narrows the awaiting case to hard-cap and circling causes, and the verb, the Go driver's escalation fact, the arbiter stencil and the refusal spec follow it.

**Commit:** 6: arbiterstop: one eligibility rule; a budget escalation goes to the parent
