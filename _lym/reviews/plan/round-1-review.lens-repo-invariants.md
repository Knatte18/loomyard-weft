```yaml
lens: repo-invariants
outcome: reported
```

Task: repo-invariants lens over cards 1–14 (seam tests, tier tags, doc text the cards falsify, card order against each card's gate).

**Per card, each invariant checked**

- **Card 1**
  - Tier tags held: `testmain_test.go` is an untagged support file, and the new tests are tier0.
  - `doc.go` held: it creates `internal/testreach/doc.go`.
  - Card order held: `internal/testreach` is not imported by any `main` yet, so the test-only-code analysis does not see it.
- **Card 2**
  - Tier tags held: tier0.
  - Doc held: it extends `doc.go`.
  - Card order held: still not imported by any `main`.
- **Card 3**
  - Doc held: it extends gatecli `doc.go` and `docs/overview.md`.
  - **Card order broken.** gatecli now imports `internal/testreach`, which puts the package inside the test-only-code guard's analysis. See F1.
- **Card 4**
  - Tier tag held: tier0 `//lym:guard`.
  - Pins held: the three pinned `ReadMap` files exist today, and card 3 creates the `WriteIndex` call before this card.
- **Card 5**
  - Seam held: no reflective test walks `gateslot.Config`.
  - **Doc broken.** The `gateslot/doc.go` bounds paragraph is not extended to the three new keys. See F3.
- **Card 6**
  - Doc held: it covers the test-speed background, gatecli `doc.go`, `docs/overview.md` and the template comments.
  - **PATTERN-gate-slot background and gateslot doc holder list broken.** See F2.
  - **Card order broken.** `ReadReachReport` and `ReachReportMatches` have no production caller until cards 7 and 9. See F1.
- **Card 7**
  - Seams held: every test marked `//lym:seam internal/shedrecipe#Env` is listed: loomcli `TestWire_EverySeamFilled`, `TestWire_DarnRecipe`, `TestWire_SamplerRecipe`, `TestWire_BeamRecipe`; battencli `TestWire_EverySeamFilled`; the envkit tests.
  - Docs held.
- **Card 8**
  - Seam held: no `//lym:seam` marker names `shuttleengine#GateEntry` or `shuttleengine#GateResult`.
  - Docs held: it extends the shuttleengine and gatefixer `doc.go` files.
  - **Card order broken.** `RerunMark.Run` and `RerunMark.On` are unreachable from any `main` until card 9. See F1.
- **Card 9**
  - Seams held: the `Env` and `Geometry` seam tests are listed.
  - Imports held: `verifytree` importing `shuttleengine` makes no cycle (`go list -deps ./internal/shuttleengine` holds no `verifytree`).
  - Docs held.
- **Card 10**
  - Tier held: tier1.
  - Docs held.
- **Card 11**
  - Tiers held.
  - **Card order broken.** `gateslot.TriageLines` is unreachable until card 12. See F1.
- **Card 12**
  - Tiers held: tier0.
  - Docs held: it extends four `doc.go` files.
- **Card 13**
  - Tier held.
  - Doc held: it extends `testcover/doc.go`.
  - **Card order broken.** `testcover.PlanLineMutants` is unreachable until card 14. See F1.
- **Card 14**
  - Tier held: tier1.
  - **PATTERN-gate-slot background and gateslot doc holder list broken.** See F2.
  - **gateslot bounds doc broken.** See F3.

**Findings**

**F1. MEDIUM, design.** Location: `_lym/plan/03-map-run-writes-index.md`, together with cards 6, 8, 11 and 13.
- **Summary:** cards create production functions whose only callers arrive in later cards. A batch gate that closes in between fails the tier1 guard `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain`, which runs at every batch gate whatever the change, because it is a `//lym:guard` test and the `batch` key runs tiers [0,1].
- **The spans:**
  - Card 3 makes gatecli import `internal/testreach`. From then until card 6, `ChangedFuncs`, `Open`, the `Reader` methods, `Fresh` and `ReadPackageMap` have no caller. `FuncsReachedBy` waits until card 10 and `ChangedLines` until card 14.
  - `RerunMark` (card 8) gets its caller in card 9.
  - `TriageLines` (card 11) gets its caller in card 12.
  - `PlanLineMutants` (card 13) gets its caller in card 14.
  - `ReadReachReport` and `ReachReportMatches` (card 6) get their callers in cards 7 and 9.
- **Evidence:**
  - `cmd/lym/testonlycode_linux_test.go:78-80` carries `//lym:guard` and `tiergate.Require(t, 1)`.
  - `pattern/PATTERN-test-only-code.md:36-38` says a package no `main` imports is outside the analysis, which is why cards 1–2 are safe and card 3 is the switch.
  - `pattern/PATTERN-test-speed.md:118` says the guard step runs every marked test.
- **Fix:** do one of these, either alone or together:
  - give each creating card its production caller in the same card;
  - merge the cards in each pair: 8+9, 11+12, 13+14;
  - move the import of `testreach` into gatecli to card 6 or later.

**F2. BLOCKING, design.** Location: `_lym/plan/14-gate-mutate.md` and `_lym/plan/06-reach-step.md`.
- **Summary:** the cards add two gate-slot holders that neither `pattern/PATTERN-gate-slot.md` nor the holder list in `internal/gateslot/doc.go` names, and no card edits either file.
  - `lym gate mutate` holds one non-tier slot through `mutationCheck.holdSlot`.
  - The reach step's refresh runs the map run's measurement inside a plain `lym gate test --gate &lt;key&gt;` call. It is not "the map run of `lym gate test --map`".
- **Why it breaks the constraint:** PATTERN-gate-slot allows a gate build to run in exactly three places: a tier run, the map run of `lym gate test --map`, or one of the non-tier holders `internal/gateslot`'s doc lists. The decision record lists PATTERN-gate-slot as a constraint.
- **Evidence:**
  - `pattern/PATTERN-gate-slot.md:12-18`.
  - `internal/gateslot/doc.go:62-67` names only the redundancy report step and `lym gate redundant`.
- **Fix:**
  - Card 14 extends `internal/gateslot/doc.go`'s non-tier holder paragraph with `lym gate mutate`.
  - Card 6 adds the reach step's refresh to `pattern/PATTERN-gate-slot.md`'s list of places, beside the map run, and to the gateslot doc.

**F3. MEDIUM, consistency.** Location: `_lym/plan/05-gate-config-reach.md` and `_lym/plan/14-gate-mutate.md`.
- **Summary:** the bounds paragraph of `internal/gateslot/doc.go` documents every bound key, says which runs take the three `redundancy_*` bounds, and says which keys `LoadConfig` refuses below 1. No card updates it.
  - Card 5 adds `reach_budget_min`, `mutation_max_mutants` and `mutation_budget_min`, but edits only the doc's `reports` paragraph.
  - Cards 6 and 14 make the reach refresh and `lym gate mutate` take the `redundancy_*` bounds, but update only the template comments.
- **Evidence:** `internal/gateslot/doc.go:174-189`, in particular line 179: "The redundancy report step of `lym gate test` and the mutation check of `lym gate redundant` take the three bounds too".
- **Fix:**
  - Card 5 adds the three keys and their refusal below 1 to that paragraph.
  - Cards 6 and 14 add their runs to the line-179 sentence.

**Tool calls**
- `grep` for `//lym:seam` markers and for "only on command" doc text, and for map-run sites in the pattern and gateslot docs. My context held no list of seam markers or of doc sentences outside the files I had read.
- Read `internal/gateslot/doc.go` lines 58-70 and 160-192, the imports of `verifytree`, the leaf-package rules and the mutation lines of `testcover/doc.go`. My context had only grep hits from that doc, not the holder and bounds paragraphs.
- `go list -deps` for `shuttleengine` and `testcover`, and the test names under each seam marker. I needed to rule out an import cycle from card 9 and to map the marker lines to the test names the cards list.
- `grep` of `docs/overview.md`, the specs and the pattern files for cold-fixer and batch-gate text. I was checking for doc text the reach turn or the fixer bypass would make false; I found none.
