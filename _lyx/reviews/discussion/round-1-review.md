---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D10 Bound)
    summary: D10 moves landing hub-wide but any pair agent can still run lyx config set landing publish_verify= from its worktree, which now rewrites and pushes the hub file for every pair; the Bound names an operator or hub-orch route that nothing enforces
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D7, Outside a hub it runs unslotted)
    summary: D7's unslotted outside-a-hub path is an escape hatch with no stated bound; it never says whether hub detection uses the cwd or the -C dir, so a cd out of the hub or a -C elsewhere can skip the slot
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D1, D5, D7, D8)
    summary: cli_wait_sec bounds one lyx gate test call, but a card gate chains one call per module inside one Bash call, so waits add up; with the 300 s default, two busy waits use the whole Bash maximum timeout, which the design was meant to avoid (Interview Q4)
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (Constraints, PATTERN-leaf-packages bullet)
    summary: the verifyrun constraint cites PATTERN-leaf-packages, which does not list verifyrun, and leaves acquisition open (told seam or callers), while the PATTERN-verified-tree constraint and the plan-writer note have already chosen callers plus an environment parameter
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (D2)
    summary: D2 says a batch gate failure counts against the self-fix cap and makes the report FAILED, with no exception for the slot-busy exit that D8 says is not a gate failure
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (D11, D12)
    summary: the Publish failure record sits under the pair's own .lyx and feeds the tier 3 round pass and the BLOCKING identities, but neither Decision says what deleting or editing it skips or that Publish's rerun is the bound
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (Acceptance criteria)
    summary: no criterion covers D7's outside-a-hub behavior, D8's slot-busy, verify-fix, plan-template and orch-stencil statements, or D11's parser fallback; the CLAUDE.md push-rule criterion matches only a Scope line, not a Decision
---

### [BLOCKING] A pair can still weaken `publish_verify`, now for every pair

**Location:** `_lyx/discussion/decision-record.md`, D10 (Decision and Bound); Goal, "a pair cannot weaken its own `publish_verify`".

**Issue:** D10 marks `landing` `HubWide` and says a landing change is "now an operator or hub-orch `lyx config set landing` at the hub".
Nothing limits that route to those two parties.
`internal/configcli/configcli.go:167` sends any `set` on a `HubWide` module to `setHubWide` (`internal/configcli/hubwide.go:129`).
That writes `<BoardDir>/_lyx/config/<module>.yaml` and commits it in `_board` from whatever worktree the command runs in.
No standing deny in `internal/shuttleengine/claudeengine/settings.go` refuses `lyx config`, and the webster stencils don't forbid it.
A pair's agent can run `lyx config set landing publish_verify=""` (or `=true`) and lyx commits it to the hub file.
Before D10 the same edit reached only the pair's own tracked copy.
After D10 it reaches every pair at once.
The Decision removes one bypass, opens a wider one, and its Bound states a restriction the code doesn't have.
The rubric's attack-surface rule applies: the guard is still bypassable, and nothing states or enforces a bound on that.

**Fix:** In D10, decide how a pair is kept from writing hub-wide `landing`.
Options: a `configcli` refusal for a `HubWide` write from a task pair (with a refusal-spec row and a test, per `PATTERN-refusal-way-forward`), or a standing deny on `lyx config set`/`edit` of hub-wide modules for spawned sessions other than the orch.
Then rewrite the Bound to name that mechanism, and add an acceptance criterion where a pair-side `lyx config set landing publish_verify=` is refused.

### [MEDIUM] D7's outside-a-hub path has no stated bound

**Location:** `_lyx/discussion/decision-record.md`, D7, "Outside a hub it runs unslotted under the template's `-p` cap and logs that no hub bound applies."

**Issue:** This is a new routing edge that skips the slot.
D7 never says what decides "outside a hub": the process cwd, the `-C <dir>`, or the `cliwire` target dir.
If it is the cwd, `cd /tmp && lyx gate test -C <worktree> ./...` runs the hub's full module unslotted.
If it is `-C`, a `-C` pointing at a copy outside the hub does the same.
D7's Bound lists the deny's static-text gaps but not this edge.
The deny can't help either: `lyx gate test ./...` is on its pass list.

**Fix:** State in D7 which directory decides hub versus standalone: the `-C` target resolved through `cliwire`, never the bare cwd.
Then add to D7's Bound what the unslotted path still lets through: a run whose target really sits outside every hub, capped only by the template's `-p`.

### [MEDIUM] The CLI wait bound doesn't fit the Bash timeout it was chosen for

**Location:** `_lyx/discussion/decision-record.md`, D1 (one `lyx gate test` step per module), D5 (`cli_wait_sec` default 300), D7, D8 ("gate commands run with the Bash tool's maximum timeout"); `support-log.md` Interview Q4.

**Issue:** Q4 picks a bounded CLI wait so that a fork's Bash call isn't killed mid-test.
`cli_wait_sec` bounds one `lyx gate test` call.
D1's card gate is a chain: one call for the root module, one per nested module, then the lint.
D2's batch gate is also one step per module.
Each step can wait the full 300 s before its tests start, so two steps hitting busy slots spend 600 s waiting.
That is the Bash tool's whole maximum timeout, before any test runs.
Even a single step leaves only the remainder for an integration-tagged batch run.
The failure Q4 meant to prevent, a Bash call dying mid-run, can still happen.

**Fix:** Pick a rule and state it in D7/D8.
Options: require a fork to run each gate step as its own Bash call; or size `cli_wait_sec` against the Bash maximum minus a stated run budget; or make the per-module steps of one gate a single `lyx gate test` invocation that holds one slot across modules.
Name the chosen relation in D5's default.

### [LOW] The `verifyrun` constraint cites the wrong pattern and reopens a settled choice

**Location:** `_lyx/discussion/decision-record.md`, Constraints, `PATTERN-leaf-packages` bullet; against the `PATTERN-verified-tree` bullet, D4 and the plan-writer note.

**Issue:** `PATTERN-leaf-packages` lists `gitkit`, `modelspec`, `buildinfo`, `standalonestate`, `segmentcolor`, `pattern` and `friction`, not `verifyrun`.
`verifyrun`'s import set is pinned only by its own package doc (`internal/verifyrun/run.go`).
The bullet also leaves "a told seam or ... its callers" open.
The `PATTERN-verified-tree` bullet says the slot is acquired inside the `verifytree.Verify` path.
The note picks callers (`verifytree.Verify` and `cardverify`), with the inheritance variable and `GOFLAGS` passed in as an environment parameter.
`verifyrun.Run(ctx, command, dir, out)` has no such parameter today.
On the focus question, D4 and the Constraints agree that environment composition happens in the callers.
Only this bullet's wording disagrees.

**Fix:** Cite `verifyrun`'s package doc instead of `PATTERN-leaf-packages`.
State in D4 that callers acquire the slot and build the environment, and that `verifyrun.Run` gains an environment parameter.
Drop the "told seam" alternative.

### [LOW] D2's failure rule doesn't carve out the slot-busy exit

**Location:** `_lyx/discussion/decision-record.md`, D2, "A failure counts as a gate failure under the existing self-fix cap and makes the report `FAILED`"; against D8.

**Issue:** On the focus question, D8 settles who handles the slot-busy code: the fork, told by its stencil.
Go never runs card or batch gates (D1, D2), and a Go-side `lyx gate test` inherits its parent's slot (D4).
But D2, the Decision that defines the failure rule, says any failure counts against the cap.
The exception for the slot-busy exit appears only in D8.
A plan writer working from D2 alone would miss it.

**Fix:** Add to D2 that D7's slot-busy exit is not a gate failure: it costs no self-fix attempt and doesn't make the report `FAILED`.

### [LOW] The failure record's tamper bound isn't stated

**Location:** `_lyx/discussion/decision-record.md`, D11 and D12.

**Issue:** The record lives under the pair's own `.lyx/`.
An agent can delete it with a plain `rm`, which isn't denied.
Deleting it drops the tier 3 round pass (D11) and the BLOCKING identities with the merge-diff widening (D12).
Editing it narrows the identities, or changes the command that D11 says the round gate reruns "under the failing command's tier".
Publish reruns both verifies regardless, so this is bounded.
D11's no-record Bound implies that for deletion, but neither Decision says the record is advisory.
Neither says the round gate builds its command from the record's fields without trusting them as a shell string.

**Fix:** Add to D11's Bound: deleting or editing the record skips only the round pass and the review widening, and Publish's own rerun stays the guard.
Also state that the round gate takes only the tier and test identities from the record, never a command string.

### [LOW] Acceptance criteria miss some observable Decision behavior

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria.

**Issue:** On the focus question, D6, D9, D13 and D14 are covered.
These are not:
- D7's outside-a-hub run, unslotted, with its log line.
- D8's statements: slot-busy is not a gate failure, the verify-fix route, the plan template's `**Verify:**` guidance, and the orch stencils' pre-push step.
- D11's fallback to the impacted-set pass when the parser extracts no identity.

The criterion "`CLAUDE.md`'s push rule names `lyx gate test`" matches a Scope line but no Decision.

**Fix:** Add a criterion for each uncovered behavior above, and fold the `CLAUDE.md` push-rule change into D8 next to the orch stencils' pre-push step.

## Focus verdicts not raised as findings

- Relocation check: `Auto-mode assumptions` restates D5's defaults, D7's unslotted package-scoped runs and D11's tier 3 condition.
  That is the section's job: recording what was picked without the operator.
  Only the `gate` segment name for the slot directory appears nowhere else, and the plan writer reads this file, so nothing is lost.
  `Open risks` holds no requirement missing from Decisions or criteria: the deny test-set risk matches the deny criterion, and the started-pair migration risk is covered by D10's board-dir load and the `publish_verify: ""` criterion.
  Not a defect.
- D10's guards: `landingshed.LoadConfig` is called at one site (`internal/loomcli/wiring.go:416`), and Publish, the PR gate and Finalize all read `require_pr_to_base`, `squash` and `publish_verify` from that one config.
  Moving the load to the board dir keeps each guard enforced at one site.
  Not a defect apart from F1.
