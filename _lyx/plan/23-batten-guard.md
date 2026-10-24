# Card 23 — batten-guard

**Create:**
- `plan:internal/battencli#ResolveBattenPrime` -> `func ResolveBattenPrime(location *lyxcwd.Location) (*lyxcwd.Location, error)`
- `plan:internal/battencli#TestResolveBattenPrime` -> `func TestResolveBattenPrime(t *testing.T)`
- `plan:internal/battencli#TestVerbs_RunFromABeam` -> `func TestVerbs_RunFromABeam(t *testing.T)`
- `plan:internal/adoptcli#TestCommand_RunsFromABeam` -> `func TestCommand_RunsFromABeam(t *testing.T)`

**Edit:**
- `internal/battencli#battenCLI.armAt`
- `internal/battencli#battenCLI.cancelCmd`
- `internal/battencli#lockStore`
- `internal/battencli#lockCommand`
- `internal/battencli#Cleanup`
- `internal/battencli#battenCLI.overviewCmd`
- `internal/battencli#Command`
- `internal/battencli#battenCLI.cleanupCmd`
- `internal/adoptcli#Command`
- `internal/cleanupcli#Command`
- `internal/battencli#TestArmAt_StartGuardRefusesBeforeAnyWrite`
- `internal/battencli#TestCancelCmd_RefusesBeforeAnything`
- `internal/battencli#stepNonPrimeRefusal`
- `internal/battencli#stepRecordsPrimeRefusal`
- `internal/cleanupcli#TestCleanupVerb_RefusesOutsideThePrimeThenListsAndThenRemovesEverySeededLeftover`
- `pattern/PATTERN-pair-teardown.md`

- `internal/adoptcli/doc.go`
- `internal/cleanupcli/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/battencli#RefuseUnlessPrime`
- `internal/fabricengine#ReadOrigin`
- `internal/fabricengine#ReadOriginFor`
- `internal/fabricengine#PrimeName`
- `plan:internal/boardengine#ReadTasks`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPairWith`

**Intent:**
D8, batten, adopt and cleanup: `ResolveBattenPrime` replaces `RefuseUnlessPrime` for every batten verb, `lyx adopt` and `lyx cleanup` (and, on card 34, every `lyx beam` verb); `lyx shed seed` keeps `RefuseUnlessPrime`, prime-only.
It runs `RequireDrivableWorktree` first, admits the prime, and admits a beam, a pair whose own origin record carries the beam mark, returning the prime's location in both cases, so each verb acts on the prime's location as today and every batten path (status file, locks, scratch, prime lock) stays under the prime's anchor.
It refuses, each naming where to run the verb:
- a beam whose board entry is run-held by its landing (`IsRunStatus` on the beam's entry), naming the landing and the beam's parent worktree, so the landing's merge fixer and conflict session reach no batten, adopt, cleanup or beam verb;
- every other worktree, a task pair that is not a beam included, naming the acting worktree, as today's refusal does, and, as where to run the verb, the nearest worktree up the pair's origin chain that exists here and is a beam, else the prime.
A verb that creates or destroys a pair refuses when the acting worktree is that pair itself: `run`, `step` and `goto` of a slug from its own pair, and `cancel <slug> --remove` from its own pair, naming the pair's parent worktree, or the prime, as where to run it; only a beam reaches this, before its landing claims, for example `lyx batten run <beam>` from inside the beam while its entry is open, since during its landing the run-held refusal fires first.
Bound: what the wider guard lets through is a session in a beam's worktree, the operator's or the beam's orch, acting on any run of the hub as from the prime; an agent of a run works in that run's own pair, which the guard refuses, and the only run whose agents work in a beam is the beam's landing, which the run-held refusal covers.
`TestResolveBattenPrime` over a hubforge hub: the prime and a marked beam both return the prime's location; a pair forked from the prime is refused naming the prime; a pair forked from the beam is refused naming the beam; a beam whose board entry holds a run status is refused naming the landing and its parent; the prime's records sibling is still refused by the drivable-worktree check.
`TestVerbs_RunFromABeam` over a hubforge hub with a marked beam `b` runs the verbs through `RunCLIIn` from `b`'s anchor: `lyx batten status <slug>` of a run seeded at the prime reads that run, arming `run <slug>` resolves the receiver's location to the prime's (so its status file and locks lie under the prime's anchor), `lyx batten cancel <unknown>` reaches the cancel's own refusal rather than the guard's, and `lyx batten cleanup` lists the prime's runs; the same verbs from a pair forked from the prime are refused naming the prime.
`TestCommand_RunsFromABeam` runs `lyx adopt <unknown>` from `b`, reaching adopt's own "not on the board" refusal, and from a non-beam pair, refused naming the prime.
`TestCleanupVerb_RefusesOutsideThePrimeThenListsAndThenRemovesEverySeededLeftover` keeps its refusal from a task pair, asserting the new text naming the prime, and gains a step that lists the prime's leftovers from a marked beam.
The integration steps assert the new text: `stepNonPrimeRefusal`'s refusal from a task pair forked from the prime still names the task worktree and names the prime as where to run the verb; `stepRecordsPrimeRefusal`'s refusal from the prime's records sibling is the drivable-worktree refusal, still naming the records sibling, wrapped in `ResolveBattenPrime`'s wording "this verb runs from the hub's prime worktree or a beam" in place of "prime worktree only", which the step asserts.
`TestArmAt_StartGuardRefusesBeforeAnyWrite` gains a row arming `run <beam>` from inside the beam, refused before any write; `TestCancelCmd_RefusesBeforeAnything` gains `cancel <beam> --remove` from inside the beam.
PATTERN-pair-teardown's Batten bookend background reads "anchored at the prime" in place of "driven from the hub's prime worktree", and the "prime only" wording of adopt's and cleanup's package docs and of `docs/overview.md`'s cleanup and adopt rows is rewritten to say the verbs run from the prime and from any beam.
Every help text and doc of the verbs this card moves that says prime-only is rewritten the same way, to say the verbs run from the prime and from any beam: batten's `Command` Long sentence "Every verb runs from the hub's prime worktree only", `cleanupCmd`'s "prime-only" doc and help, `lockCommand`'s Long "Both verbs run from the hub's prime worktree only", `cancelCmd`'s doc "the prime-only end of a run for good", `overviewCmd`'s Long "runs from the prime only", and the `Command` Long of `adoptcli` and of `cleanupcli`, "It runs from the hub's prime worktree only".

**ImpactSummary:** Seven guard call sites move to the beam-admitting guard; shed seed keeps the prime-only one, and the run and cancel verbs gain a self-pair refusal.
