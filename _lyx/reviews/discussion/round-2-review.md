---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D8 (card Verify runner bullet), D13, PATTERN changes
    summary: A Webster fork runs each card Verify line itself in a foreground Bash call, so a line that is not a lyx gate test call, such as the go vet ./internal/x the acceptance criteria use, runs with no slot, -p or -trimpath, against the declared PATTERN-gate-slot text
    fix: In D8 and D13, add a decision for the fork's own run of a card Verify line, not only the Go re-run in rerunCardVerifies. Either (a) the fork prompt has the fork run a line that is not one lyx gate test --wait call through a slot-holding lyx verb, which D13 then lists as a non-tier holder with its slot, timeout and GOFLAGS, or (b) the plan gate refuses a Verify line that is neither one lyx gate test --wait call nor free of go build, go vet and go test in command position, and D10 says so. Whichever is picked, add an acceptance criterion that the fork's run of a go vet Verify line holds one slot, or is refused at the plan gate. If neither fits, amend the declared PATTERN-gate-slot text in D13 and PATTERN changes so it names the fork's own run of a Verify line as an exception, with its bound.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D8 (verifytree bullet and Publish failure bullet)
    summary: D8 says every command verifytree.Verify runs is one lyx gate test --wait call, but the confirm-the-fix pass prepends the failing tests and appends a package pass to the round command with &&, and today renders those as raw go test
    fix: In D8's Publish failure bullet, state that the confirm-the-fix pass renders each failing-test run and the package pass as its own lyx gate test --wait --tier <failing tier> call, the failing tests as explicit targets with -run as the words after --, joined to the round's gate call with &&. Reword D8's verifytree bullet to say every command it runs is one lyx gate test --wait call, or such calls joined by &&, so it holds no slot itself.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria, third bullet
    summary: The criterion that a command without {targets} runs no command for a .md-only diff contradicts D3 under the none scoper, where a computed scope is whole and the command runs
    fix: Reword the third acceptance criterion's tail to read, under the go scoper, for a .md-only diff runs no command, like every other tier; under none, a .md-only diff runs every listed tier whole.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D13
    summary: D13 says cmd/testtiming's measured go test holds one slot, but only its -resources mode takes one; its default timing mode and -redundancy mode run go test over packages unslotted, and the record neither slots them nor states them outside PATTERN-gate-slot
    fix: In D13, decide for each cmd/testtiming mode. Either every mode takes one slot and appends -trimpath and -p as -resources will, or D13 states that only -resources is a gate holder and the timing and -redundancy modes are operator measurement runs outside PATTERN-gate-slot. In the second case, amend the declared PATTERN-gate-slot and PATTERN-test-tiers texts to name -resources, or every mode, matching the decision.
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D1 (flow-mapping --set), D12
    summary: D12's tier values hold {targets}, {args} and {parallel} inside a YAML flow mapping, where a plain scalar may not contain a brace, so the orch's --set written as D12 spells it fails to parse
    fix: In D1's flow-mapping sentence, add that a value holding a brace, comma or bracket is double-quoted inside the mapping, and change the example so its command value is a double-quoted string holding {targets} and {args}. In D12, write the GOFLAGS value of the env entry as a double-quoted string, so -trimpath -p={parallel} parses.
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md Open risks, first bullet
    summary: An in-flight plan's card Verify line written for today's gate, holding --tags or lacking --wait, passed the old plan gate, and after the deploy the card re-run fails it on the --tags refusal; Open risks names only fork prompts
    fix: Extend Open risks' first bullet with a sentence that an in-flight plan's card Verify line holding --tags fails its card re-run after the deploy, and is fixed by rewriting the line to --tier with --wait.
---

### [BLOCKING] The fork's own run of a card Verify line holds no slot

**Location:** `_lyx/discussion/decision-record.md` D8 (card `**Verify:**` runner bullet), D13, `## PATTERN changes`

**Issue:** D8 and D13 bound only the Go re-run of a card's `**Verify:**` line: `internal/websterengine/cardverify.go`'s `rerunCardVerifies` takes one slot for a line that is not a gate call.
The line is also run by the implementer fork itself.
`contracts/stencils/webster/webster-body-implementer.md` step 5 tells the fork to run each line of a card's `verify:` "as its own foreground Bash call under the gate rule", and nothing in that path takes a slot.
D10 refuses only `refuse_raw` prefixes (`go test` on loomyard), and the shell guard's `raw-go` row (`internal/shellguard/rows.go`, `rawGoCommandPattern`) lets any package-scoped `go build` or `go vet` through.
So `go vet ./internal/x`, the line the acceptance criteria name, runs in the fork's Bash unslotted, with no `-p` cap and no `-trimpath`.
The declared `PATTERN-gate-slot` text allows a gate build or test only inside a tier run or "in one of the non-tier holders `internal/gateslot`'s doc lists, holding one slot".
The fork's run of the card's gate is neither, so this task's diff would make its own declared text false.
Under the rubric's constraint rule that is BLOCKING.

### [MEDIUM] Confirm-the-fix is not one gate call

**Location:** `_lyx/discussion/decision-record.md` D8

**Issue:** D8 says "`verifytree.Verify` no longer takes a gate slot ...: every command it runs is one `lyx gate test --wait` call".
The round's confirm-the-fix pass in `internal/loomshed/verifygate.go` builds `failedTests + " && " + runCommand` and appends the package pass with `&&`.
`internal/loomshed/publishfailure.go`'s `publishFailureSteps` renders both as raw `go test -tags ... -run ... <pkg>`.
D8 says only that the pass "runs that tier for the record's failing tests and packages".
It never says each step becomes a slotted gate call.
Read literally, the verifytree bullet rules out the `&&` composition.
If the plan writer keeps today's raw steps, they run unslotted once verifytree drops its slot.

### [MEDIUM] Acceptance criterion contradicts D3 under `none`

**Location:** `_lyx/discussion/decision-record.md` `## Acceptance criteria`, third bullet

**Issue:** The bullet says a command without `{targets}` "for a `.md`-only diff runs no command, like every other tier", with no scoper named.
D3 makes a computed scope under `none` whole, with `{targets}` taken from `all_targets`, and only the `go` scoper yields an empty set for a `.md`-only diff.
NORCE Models' tiers all lack `{targets}` and run under `none`, so this criterion and D3 give opposite outcomes for the same call.
The rest of the `none` statements in D1, D2, D3 and D7 agree with each other: explicit targets pass verbatim or run whole without `{targets}`, words after `--` go to `{args}`, and no zero-test or floor check applies.

### [MEDIUM] `cmd/testtiming`'s unslotted modes

**Location:** `_lyx/discussion/decision-record.md` D13

**Issue:** D13 lists "`cmd/testtiming`'s measured `go test`" among the paths that "hold one gate slot".
In code, only the `-resources` mode takes a slot (`cmd/testtiming/resources.go`, `takeGateSlot`).
The default timing mode (`runPackageTimings`) and `-redundancy` (`go test -c`, `go test -list`) run `go test` with no slot, no `-p` from the pool and no `-trimpath`.
The declared `PATTERN-test-tiers` text exempts "`cmd/testtiming`'s measured run".
The declared `PATTERN-gate-slot` text needs that run to hold one slot.
So the record leaves open whether two of the three modes break the declared slot text or sit outside it.

### [LOW] Flow-mapping values need quoting

**Location:** `_lyx/discussion/decision-record.md` D1, D12

**Issue:** D1 has the orch write each tier whole as a YAML flow mapping, and D12 spells loomyard's values for it.
Those values include `env: {GOFLAGS: -trimpath -p={parallel}}` and commands ending `{targets} {args}`.
In YAML flow context a plain scalar cannot contain `{`, `}`, `[`, `]` or `,`, so these values fail to parse unless quoted.
The loader would catch the failure, but the record presents the values as ready to apply.

### [LOW] In-flight Verify lines after the deploy

**Location:** `_lyx/discussion/decision-record.md` `## Open risks`

**Issue:** The first Open risks bullet names fork prompts rendered with `--tags`.
A plan already past today's plan gate can hold a card `**Verify:**` line such as `lyx gate test --tags integration ./internal/x`.
After the deploy, `rerunCardVerifies` runs it, D2 refuses `--tags`, and the card fails its audit with no stated way forward.

## Focus departures

None.
The four directives were each reached.
On the tree-scoped verify (directive 1), D9 makes `tree_scope_verify` one `lyx gate test --wait` call with no shell control operator, so it is slotted by the gate, and that is not a defect.
On darn (directive 3), Merge-In, Finalize and Publish of a tree-scoped diff run `tree_scope_verify`, and Publish skips `publish`.
That matches today's wiring (`internal/loomcli/darnwiring.go`, `darnVerifySource` and `publishSkip`).
A run with an empty rendered `{targets}` runs no command, which `PATTERN-verified-tree` does not forbid, and that is not a defect.
On D5, D6 and D12 (directive 4), the guard step's slots and timeout are `guard_tier`'s.
Loomyard's tiers get `-trimpath` and `-p` through each tier's `env`.
The prebuild passes both flags explicitly (`internal/gatecli/prebuild.go`).
The non-tier holders append both, so the only gaps are F1 and F4.
