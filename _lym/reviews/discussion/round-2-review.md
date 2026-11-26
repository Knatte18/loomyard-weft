---
verdict: BLOCKING
findings:
  - id: F20
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:289
    summary: "D13's seam meanings break every test binary outside fabric. cmd/lym/main.go installs seams only under !testing.Testing() (main.go:162; prerunseams_test.go:2), and the record gives only fabric's own tests a way to supply them. Every package's tier1 tests build pairs through hubforge.AddPair, which runs Topology.Add (hubforge/hub.go:483). Add commits the run-records root (add.go:709-710), and Remove, pull, replay and the origin reconcile use run paths, run locks and shed's codec. So those suites reach the required-seam refusal. The arbiter entry freezes hubforge's import set, so hubforge cannot install shed's adapter. The nil workspace builder likewise stops writing the hub workspace file that beamcli (dobbyleftover_integration_test.go:119), ideengine (spawn_clean_integration_test.go:193) and machinecli (start_test.go:197) tests read today with no hook installed."
    fix: "In D13, after the required-and-optional seams list, add a decision saying how a test binary outside fabric gets the seams, within the arbiter's bound on hubforge's import set. One option: put a single installer in the wiring package, internal/fabriccli. It installs the real shed adapter for run-records paths, run locks and codec, and the stencil judge. cmd/lym/main.go calls it in place of its separate installs, and hubforge, which already imports fabriccli, calls it when it builds a hub. That adds no hubforge import and keeps the installer reachable from main. Another option is an install every such test binary's TestMain calls. For the workspace builder, say what the beamcli, ideengine and machinecli tests do: install the IDE module's builder, as ideengine/spawn_hub_integration_test.go does today, or assert over fabric's own state. Then extend D5's sentence on test binaries to name this, and widen the cmd/lym seams criterion, or add one, so a test outside fabric that builds a pair through hubforge runs unrefused."
  - id: F21
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:265
    summary: "D13's shed-write sequence reads the record's bytes, has the seam's encoder return new bytes, then writes them under the lock. Today both writes run state.WriteJSON (codetipsettle.go:90, takeoverwrite.go:228), which takes the status lock first and reads, merges unknown keys and writes under it (state.go:35-41, 77-99). The new order opens a lost-update window against shed's own state.UpdateJSON under StatusLock (shedrun/intents.go:22). It also moves the rewrite out of internal/state, breaching the listed PATTERN-tolerant-records ('a rewrite goes through internal/state'). And 'the encoder is shed's own writer' does not give today's bytes: the takeover write is a one-key patch merge (takeoverwrite.go:58-62), not a full-Status rewrite."
    fix: "Rewrite D13's 'Writes' sub-bullet. Fabric takes the status lock at the path the seam names. Under that lock it reads the current file, applies the change, and writes through internal/state's merge, so unknown keys are kept as today. The takeover write stays a patch of only the takeover key, through state.WriteJSON with fabric's own takeover entry type, whose JSON shape matches today's; it needs no codec. The code-tip settle asks the seam's decoder and encoder for the rewound status from the previous transition's bytes, and fabric merges the result over the current file through internal/state under the same lock. Then extend the same-bytes acceptance criterion: the status file holds a key shed's type does not declare, and the bytes after a takeover write and a code-tip settle equal today's, that key included."
prior:
  - id: F1
    status: resolved
    answer: "D12 defines 'above fabric' by dependency closure, D13 leaves internal/fabriccli unbound, and every seam, both detached children's included, is installed at process start by cmd/lym/main.go."
  - id: F2
    status: resolved
    answer: "D13 decides how each write and reader crosses the cut, and an acceptance criterion checks the bytes. The write mechanism's own defect is the new F21."
  - id: F3
    status: resolved
    answer: "D13 replaces the bytes with an installed workspace builder, called on every regenerating exit, and states its nil and failure meaning. The nil meaning's effect outside fabric is in the new F20."
  - id: F4
    status: resolved
    answer: D2 lists the handed facts, the parent verdicts and the child's core rules, keeps the pending record's shape, and binds the in-process path.
  - id: F5
    status: resolved
    answer: D3 names fabric-defined outcome classes with the mapping in the engine, a wait action, and the archive tag push's inner retry kept in one action.
  - id: F6
    status: resolved
    answer: D3 and D4 end the list on failure or refusal, compute the inverse list over executed actions and pre-creation facts with the engine keeping the tokens, name the effects with no inverse, and keep the best-effort rule.
  - id: F7
    status: resolved
    answer: D4 records exactly today's entries kind for kind, and an acceptance criterion checks the printed mutations array.
  - id: F8
    status: resolved
    answer: D1 puts MutationRecord in the layout as plain data with the recorder in the engine, and D7 and D9 move shed-carrying result types in the part that cuts their shed import.
  - id: F9
    status: resolved
    answer: D7's rationale is narrowed to importers whose tests reach no engine importer, and D10 counts the packages that left the gate scope.
  - id: F10
    status: resolved
    answer: D12 admits only kits none of whose imports is above fabric save hubforge, names boardkit, envkit and takeoverkit as outside the set, and the guard criterion fails on such a kit import.
  - id: F11
    status: resolved
    answer: "D2 records go-git v5.19.1's verifyExtensions refusal of refstorage at git.Open, pinned to repository_extensions.go. That holds through gitrepo's config wrapper, which keeps Raw (gogit.go:173), and the fallback and D6's test cover refs as well as worktrees."
  - id: F12
    status: resolved
    answer: D13 states the required and optional seams, the refusal of an unset required seam, and its acceptance criterion.
  - id: F13
    status: resolved
    answer: D10 measures before and after on one machine in one session, takes the median of at least two runs, records load and held slots, and judges a relative cut.
  - id: F14
    status: resolved
    answer: D12 decides that a core row sets config-derived values as fabric's own state and that engine tests load real config.
  - id: F15
    status: resolved
    answer: D9 lands the PATTERN-fabric-core entry and the PATTERN-fabric-boundary rule line in part 7 beside their guards, and each other edit where it first holds.
  - id: F16
    status: resolved
    answer: D1 names both new packages' closed import sets.
  - id: F17
    status: resolved
    answer: D2 names the teardown child and the SpawnDetachedPush child, and places the background pusher in internal/runpusher.
  - id: F18
    status: resolved
    answer: "Settled by the added entry 'push-retry-not-halt waits for the whole chain'. That entry has the writer drop Open risks 1 and 2 and mark Q12 answered, and both still stand in the record and the ledger."
  - id: F19
    status: resolved
    answer: "Settled by the added entry 'hubforge, boardkit and envkit keep their import sets in this task'. That entry has the writer drop Open risk 3 and mark Q15 answered, and both still stand in the record and the ledger."
---

### [BLOCKING] D13's seams break every test binary outside fabric

**Location:** `_lym/discussion/decision-record.md:289` (D13, required and optional seams), with D5 at :135-136.

**Issue:** `cmd/lym/main.go` installs every fabric seam inside `if !testing.Testing()`, so no test binary has one.
D13 makes the run-records paths, run locks and shed's codec required and has an unset one refuse before acting, but it says only how fabric's own tests supply them.
Every package whose tier1 tests make pairs through `hubforge.AddPair` reaches `Topology.Add`, which commits the run-records root.
Those packages' tests drive Remove, pull, replay and the origin reconcile too, so the existing suites across the repository meet the refusal.
The arbiter's entry keeps hubforge's import set, so hubforge cannot install shed's adapter by importing it.
The nil workspace builder has the same cause.
Today a regeneration with no hook still writes the member-less file through `vscode.BuildWorkspace` (`hubworkspace.go:123-126`, `:212`).
Under D13 it writes none, so the beamcli, ideengine and machinecli tests that read `HubWorkspacePath` fail or pass vacuously.
All four lenses raised this, and it is the same defect found from four directions.

### [BLOCKING] D13's shed-write sequence drops `internal/state`'s locked merge

**Location:** `_lym/discussion/decision-record.md:265` (D13, the shed records "Writes" sub-bullet).

**Issue:** Today the code-tip settle and the takeover write both run `state.WriteJSON`.
It takes the status lock, reads the file as it stands, keeps the keys its type does not declare, and writes, all under that lock.
D13 reads the bytes first, encodes, then writes under the lock.
A shed write between the read and the write is lost, and the rewrite no longer goes through `internal/state`, which the listed `PATTERN-tolerant-records` requires; a listed constraint's breach is BLOCKING.
The claim that shed's own writer keeps today's bytes fails for the takeover write.
That write merges a one-key patch over the current file, while shed's writer re-marshals a whole `shedengine.Status`.
The lenses rated this MEDIUM; it is raised to BLOCKING for the constraint breach.

## Focus answers

- Module cut: fabric's production closure holds the four cut modules and otherwise only packages below fabric by D12's rule (`go list -deps`). The closure guard can fail, on any path to a cut module. Both children get their seams from the root's PersistentPreRunE, which runs for hidden verbs. The gap outside the real binary is F20.
- Shed records across the cut: the readers hold (bytes plus the caller's decode). The writes do not (F21).
- Seams unset or failing: F20.
- Core and executor split: the push classification, the wait action, the archive tag push's inner retry, a mid-list failure or `destroy.go` refusal, the rollback tokens and the mutations array are all placed, and verb behaviour holds.
- Detached teardown: the handed fields, the parent verdicts and the child rules match `remoteTeardownJob`. The pending record keeps its shape, and the in-process path follows the same rule.
- Part ordering: each part can land under the text and guards in force then. Result types embedding `MutationRecord` stay in the engine and embed the layout's plain type; shed-carrying ones move in the cutting part.
- Tests and speed: the admitted kits hold. The gate-scope claim is narrowed correctly for hubforge, boardkit and envkit. The load protocol records load and does not control it (below, not raised).
- Unpinned tool behaviour: pinned and correct (F11).

## Rejected

None of these is a false positive.
Each is a real defect below BLOCKING, and an incremental round raises only BLOCKING findings:

- Open risks 1 to 3 and the ledger's Q12 and Q15 still stand after the arbiter's entries told the writer to drop them (interactions 5); the F18 and F19 answers name them.
- The declared `PATTERN-fabric-boundary` text counts `fabriccli` as fabric while D12 counts it above fabric (interactions 4, bound-breaker F-e).
- The declared rule line "only `fabricengine` runs git" leaves out the background's warp-or-weft scope and read-only exemption, and fabriccli's read-only git would breach it (bound-breaker F-d).
- In effect the D13 closure guard catches only the four cut modules and what reaches them (bound-breaker F-f).
- The takeover write's `Delays` option and `time.Sleep` differ from D3's single delay source and `BackoffSleep` (bound-breaker F-g).
- D10 records load at each run's start but does not interleave the runs or bound the load (walkthrough F-W4).
