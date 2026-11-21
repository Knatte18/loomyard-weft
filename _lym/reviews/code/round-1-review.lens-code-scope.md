```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced the diff's changes to the decision record's goal points and its Added-after-Discussion entry.

**Decisions**

The `## Goal` body is corrupted, so its surviving goal points are the three triage bullets. The one `Added after Discussion` entry, "Scope after the corrupted board body", restates them as (1) #173, (2) #175 and (3) #174.

- **D1. Red-first (triage bullet 1, Added entry (1), #173):** an unjudged or no-test refusal does not spend a verify attempt, and the gate waits for the parent's decision.
  - Built by C1.
- **D2. Way forward (triage bullet 2, Added entry (2), #175):** a stencil build-version refusal at Preflight carries only its own way forward.
  - Built by C2.
- **D3. Missing findings path (triage bullet 3, Added entry (3), #174):** reproduce what removes the file, then fix that or make the halt name a path that survives.
  - Built by C3. The commit message and summary name the cause: `clearGateFindings` deleted the file on a later pass, reproduced from the raddle-slug-length-check trace. The diff removes that call.
  - The entry offers the orphan sweep only as a suspect, and it lets the writer fix whatever removes the file. Not touching `internal/battencli/cleanup.go` is therefore no scope gap.

**Changes**

- **C1. Red-first hold (commit 021ac38d): traces to D1.**
  - Production code: `internal/loomshed/redfirstgate.go`, which adds `RedFirstGate` with Gate and Final, the hold, `holdsForCommitOrRecord` and `redFirstHoldLine`.
  - Forced by C1:
    - the HeadTree export in `internal/verifytree/verify.go` and `verify_integration_test.go`;
    - the MayHold and Final wiring in `internal/shedrecipe/entries_gate.go`, which `evaluateGate` (`internal/shuttleengine/wait.go:1834`, `:1878`) needs;
    - the doc updates in `internal/loomshed/doc.go` and `internal/shedrecipe/doc.go`;
    - the comment in `contracts/recipes/darn-recipe.yaml:139-141`;
    - the stencil wording in `contracts/stencils/darn/darn-template-write.md:110-112`, where the old text said the gate fails on these exits;
    - the line in `docs/overview.md:1110`;
    - the tests in `redfirstgate_test.go` and `entries_gate_test.go`.
- **C2. `preflightTableError` (commit 445885b0): traces to D2.**
  - Production code: `internal/loomcli/darnwiring.go:357-377`, plus the `wireDarn` doc line.
  - Forced by C2: `TestPreflightTableError_…` and the header tweak in `wiring_darn_test.go`.
- **C3. Keeping the findings file after a later pass (commit 4634ff67): traces to D3.**
  - Production code: `clearGateFindings` and its three call sites are removed from `gateattempts.go`, `burlerfix.go`, `burlerreview.go` and `multillm.go`.
  - Forced by C3:
    - `TestClearGateFindings` is deleted with the function;
    - the "removed" tests in burlerfix, burlerreview, multillm and `entries_burlerhalves` become "kept";
    - `latestDarnOutcome` names the file only on a Stuck. Without that, the prior-work note would point a spawn after a pass at old findings, since the file is now kept;
    - the doc comments in `shedadapters/doc.go`, `BurlerHalfDeps.GateFindingsPath`, `KeepGateFindings` and `gateFailedReason`.
- **C4. Comment rewrap and the alignment of a stale findings-file test (commit a6cbb172): forced by C1 and C3.**
  - It reflows comments those commits added and aligns a test C3 made stale. It adds no new behaviour.

**Findings**

None. Every change traces to D1, D2 or D3, or is an edit a traced change forces, and every decision has a change that builds it.

Outside my lens: the C3 change leaves the `shedrun.GateFindingsFile` doc stale. At `internal/shedrun/paths.go` it still says "the row's next call whose gates pass removes it". That belongs to the affected-set lens.

I made no tool calls.
