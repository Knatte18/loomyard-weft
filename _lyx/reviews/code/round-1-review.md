---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/decision.go:751
    summary: a spawned role's parent name and the loom decision guard still follow the recorded parent via ResolveParent, while notices follow the nearest live orch
    fix: In internal/loomcli/decision.go (the parent resolution at line 751 and the caller check at line 565) and internal/loomcli/wiring.go:390 (the Discussion-Write parent-review addressee), resolve the parent through the nearest-live-orch walk (hubgeom.NearestLiveOrch with the orch liveness predicate) wherever the answer names the orch that receives the run's notices, keeping ResolveParent only for the parent pair itself. If the record's accepted staleness of the driver's parent message is meant to cover these readers, instead state in internal/hubgeom/doc.go that parent names for prompts and the decision guard keep the recorded parent.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/add.go:506
    summary: Add over an adopted weft branch that already has an origin record writes no record, so Beam, Parent and ParentBranch from AddOptions are dropped silently and beam create reports the beam unmarked until a re-run
    fix: In internal/fabricengine/add.go at the origin-write condition, when the record exists and opts.Beam is true or opts.Parent is set, rewrite the record with the options' Beam, ParentBranch and ParentWorktree in the same origin commit; alternatively have createSteps in internal/beamcli/create.go call deps.MarkBeam after a PairAdded result so the mark is always set in the create.
  - id: F3
    severity: LOW
    class: consistency
    origin: handler
    location: internal/boardengine/rename_test.go:20
    summary: the word umbrella is used for the project fixture slug, title, variables and messages in added test code, where the record requires beam throughout and never umbrella
    fix: In internal/boardengine/rename_test.go replace the fixture slug "umbrella" with "grouping" and "umbrella-two" with "grouping-two", the title "Umbrella" with "Grouping", the local variable umbrella with grouping, and update the doc comments and the t.Errorf and t.Fatalf messages that name it.
  - id: F4
    severity: LOW
    class: consistency
    origin: handler
    location: docs/overview.md:628
    summary: the batten paragraph still says lyx batten run --window opens a window of the orch's reed session, while D8 opens it in the run's parent worktree's session, the prime's or a beam's
    fix: In docs/overview.md reword the sentence "`lyx batten run <slug> --window` starts the same run in its own tmux window of the orch's reed session" to say the window opens in the reed session of the run's parent worktree, the prime's or a beam's, with the prime as its working directory.
---

### [LOW] Parent names and the decision guard keep the recorded parent

**Location:** internal/loomcli/decision.go:751
**Issue:** D13 sends notices up the chain to the nearest live orch and keeps `ResolveParent` for readers that need the parent pair itself.
The diff moves the notice readers, but a member's agents are still told the recorded beam orch is their parent, and `lyx loom decision add --by parent` compares the caller to that name.
With the beam's orch stopped, the prime's orch receives the notices and the guard then refuses it.
D13's enumerated readers do not list these callers and the record accepts a stale driver parent message, so this is LOW.

### [LOW] Add drops the beam mark over an adopted weft with a record

**Location:** internal/fabricengine/add.go:506
**Issue:** The origin record is written only when the weft was not adopted or has no record.
A beam create over such a branch gets no mark, and the envelope reports the beam unmarked.
The next create re-run repairs it through `MarkBeam`, so the damage is bounded.

### [LOW] Umbrella in the rename test fixture

**Location:** internal/boardengine/rename_test.go:20
**Issue:** The fixture, its comments and its messages use the word umbrella, which the decision record's Goal forbids in every name and text the task touches.
The lens reports of the pattern-fit and affected-set forks raised the same occurrence; they are merged here.

### [LOW] Stale window-session sentence in the overview

**Location:** docs/overview.md:628
**Issue:** The diff updates the verb's help text and the Worktree-Create paragraph, but the sentence on `--window` still names the orch's reed session.

## Rejected

- code-tests: `BeamWithMembersIsRefusedNamingEachOpenEntry` duplicating `TestBeamLandRefusal` — the fork did not confirm what the integration test asserts, and a redundancy finding needs a named covering test the new one repeats; it pins the guard wiring through a real hub.
- code-tests: no named test for `refuseOwnPair` on `goto` — the fork recorded it as minor and not a finding; the guard shares one code path with `run` and `step`.
- code-tests: `TestEndRunForPromote` and `TestClearRunForPromote` as duplicates of the cancel tests — they pin the promote-specific re-run text and the end-only error.
- Holistic: the declared PATTERN changes both land with their declared text, so no declared-change finding is raised.
