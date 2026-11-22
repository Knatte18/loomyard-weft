---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/gitrepo/pull.go:43
    summary: Unrestricted fetch pruning deletes local recovery tags in one-repository records stores.
    fix: Restrict pruning to remote-tracking branch destinations while retaining local tags and fetching namespaced remote tags without pruning them. Keep namespace interpretation in fabric. Add a regression with the records-store refspecs, a deleted remote branch, a local-only set-aside tag and a live remote tag; assert that the deleted branch tracking ref disappears, the local tag survives and the remote tag is fetched.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/runfsm/act_test.go:70
    summary: No refusal assertion covers an empty code tip or the pair-absent machine-absent variant.
    fix: Add a TestAct table row with CodeTipMissing and CodeTipPairAbsent true and CodeTip and CodeTipMachine empty. Assert the complete generic reason and fetch-first way forward with the machine-free fallback.
    origin: lens:code-tests
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/battencli/arm_runverdict_test.go:25
    summary: The refresh test cannot exercise an error from the second read.
    fix: Let the existing table supply errors per read and add a row whose first read returns a code-tip wait, whose fetch succeeds and whose second read returns a sentinel error. Assert that error, one fetch and two reads.
    origin: lens:code-tests
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lym/landing/summary.md#Merge danger
    summary: The two-way classification and gitrepo-only blast radius omit destructive ref pruning and fabric recovery consumers.
    fix: Update Merge danger to classify the deletion of refs as a one-way change and name the code and records stores, fabric recovery tags and batten start checks in its blast radius. Describe the final implementation after fixing F1; do not retain the claim that reverting the binary restores deleted refs.
    origin: handler
---

### [BLOCKING] Preserve local recovery tags during fetch

**Location:** internal/gitrepo/pull.go:43

**Issue:** The records store configures `+refs/lym/tags/*:refs/tags/*` in `internal/fabricengine/clone.go:829`.
Git's installed fetch manual states that pruning applies to explicit refspec destinations, including local tags.
The configured `--no-tags` disables automatic following, not this explicit mapping.
`RemoveLocal` retains removed records through a local set-aside tag (`internal/fabricengine/remove.go:216–225`).
Its creation reaches `setAsideTag` in `internal/fabricengine/replay.go:527–560` and pushes nothing.
A subsequent records fetch deletes that recovery tag because origin has no matching namespaced tag.
The promised recovery reference disappears, leaving otherwise unreachable commits eligible for collection.
Both correctness and affected-set lenses independently identified this defect.

### [MEDIUM] Assert the generic refusal

**Location:** internal/runfsm/act_test.go:70

**Issue:** Every code-tip refusal row supplies a nonempty tip and overwrites the default reason at `act.go:202`.
The supplied mutation report's surviving deletion therefore corresponds to an observable, unasserted message.
The pair-absent row also always names a machine; the machine-free fallback is asserted only with a pair present.

### [MEDIUM] Cover the refreshed read's failure

**Location:** internal/battencli/arm_runverdict_test.go:25

**Issue:** The single `readErr` applies to every read, so an error ends the test before it reaches the fetch.
The new second read at `arm.go:665` has no case asserting that its error propagates.
The existing successful-refresh, unresolved-tip, failed-fetch and non-code-tip rows use independent expectations.

### [MEDIUM] Record the deletion risk and affected callers

**Location:** _lym/landing/summary.md#Merge danger

**Issue:** The description calls the change two-way and names only gitrepo as its blast radius.
The diff deletes tracking refs, and its configured tag mapping also deletes the recovery refs described in F1.
Reverting the executable does not restore deleted references.
The effects reach fabric's records recovery and batten's permission to start a run.
The rubric explicitly requires a finding for a two-way classification of a change that deletes data.

The fetch path reaches the prime's records worktree when the pair is absent and both pair repositories otherwise.
The second judgement invokes ReadRun again; OriginCodeTip creates new repository handles and resolves refs again.
TestReadRun pins both values of CodeTipPairAbsent.
The scope permits the records-fetch alternative, so leaving cancel --remove unchanged is not a scope defect.
Both Root-Cause trailers name defects removed by the diff.

Validation attempt: the tier-0 gate could not acquire its shared slot because the hub's gate directory is read-only.
No tests ran in this review attempt; the supplied gate reports were considered as evidence, not rerun results.
