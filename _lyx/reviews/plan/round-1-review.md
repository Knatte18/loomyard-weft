---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/12-publish-push-rejected.md
    summary: Card 12 departs from the record's drive-the-real-CLI-wiring constraint on a premise that a loom-step harness is missing, but loomcli's wire()-driven tests already give one
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/10-fabric-remote-only-commits.md
    summary: ImpactSummary says RemoteOnlyCommits has two consumers in other packages, but the reset consumer sits in fabricengine itself
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/plan/11-reset-moves-remote-branch.md
    summary: The remote update ignoring SkipGit is a choice the record leaves open, and the Intent justifies it by the record's silence without stating what bounds it
---

Fidelity is complete: D1 to D10 map to cards 1 to 8 and 9 to 12, and every Constraint has an owner.
The refusal-spec rows for D4 to D8 sit in cards 7, 5, 6, 11 and 12 and D10's row removal in card 4.
The `PATTERN-mutation-record` partial envelope is in card 11 and the plan-check rows are in cards 1, 2 and 3.
The destructive-guard tally rewording in card 11 is required hygiene, since the new executor and token make the pinned counts false.
It adds no scope.
The `## verify:` section runs `go test ./...` untagged and the `integration` tag over every package a card targets that holds tagged tests (`tools/sandbox` and `claudeengine`'s `modelswitch_test.go` are untagged).
Card 11 is one unit: the executor, the guard test and the CLI envelope cannot land separately without breaking `destructiveguard_test.go` or the `PATTERN-mutation-record` partial rule.
Every test named in the cards exists in the tree.

### [MEDIUM] Card 12 skips the CLI-wiring regression on a false premise

**Location:** `_lyx/plan/12-publish-push-rejected.md` and the `cli-drive` Shared Decision in `00-overview.md`.
**Issue:** The decision record's board instruction is that each fix's regression drives the real CLI wiring, and its last acceptance item repeats it.
The plan departs for D8 because "no loomcli test steps the Publish row today, so driving it through `lyx loom step` needs a new loom-step harness".
`internal/loomcli`'s `wiring_rework_integration_test.go` already drives `wire()`'s real seams over a hubforge pair, and `wire()` assembles `c.env.Landing` through the production `landingDeps`.
A test can build `landingshed.NewPublish(c.env.Landing)` on such a pair and reach the rejected half, which halts before any GitHub call, with no new harness.
Only the resumed half needs the GitHub fake, and `landingshed`'s integration test keeps that.
The new `landingdeps_integration_test.go` as planned asserts only the closure's return value, which `fabricengine`'s own planned test already pins.
**Fix:** Replace the Create target with a `wire()`-driven integration test in `internal/loomcli` that steps Publish over a hubforge pair whose remote task branch holds a commit the local branch lacks.
It asserts the stuck reason names the remote tip, the count of 1 and the merge-then-`lyx loom start` way forward, through the production closure.
Reword the `cli-drive` rationale to say the rejected half is driven through `wire()` and only the resumed half stays at `landingshed`'s tier.

### [LOW] Card 10 ImpactSummary misstates its consumers

**Location:** `_lyx/plan/10-fabric-remote-only-commits.md`, `**ImpactSummary:**`.
**Issue:** "two consumers in other packages" is false for the reset consumer, `ResetPairWarp` in `internal/fabricengine/destroy.go`.
The count is also a perishable tally of consumers.
**Fix:** State the conclusion without the count or the package claim: a new exported read, read-only apart from the fetch's remote-tracking refs, no existing symbol changes.

### [LOW] Card 11 states no bound for ignoring SkipGit

**Location:** `_lyx/plan/11-reset-moves-remote-branch.md`, the paragraph on `SkipGit`.
**Issue:** `SyncOptions.SkipGit` is documented as skipping all git operations, and the record names only `SkipPush` as a bypass without saying what `SkipGit` does.
The card resolves it by letting the remote update run under `SkipGit`, and its only stated reason is the record's silence.
The attack-surface rule wants what the step lets through and what bounds it.
**Fix:** Add to the Intent that the update ignores `SkipGit` and is bounded by the same gate, ancestry check and lease as without it.
It rewrites only the pair's own task branch to the reset commit, and only when the remote holds nothing the local HEAD lacks.
