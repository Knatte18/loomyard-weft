---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md:174 (D8 last bullet), :198 (D10 plan-gate rule)
    summary: The card Verify runner loses its gate slot, yet only refuse_raw prefixes are refused in a Verify line, so a Verify line such as go build ./..., go vet ./... or a test script runs as an unslotted gate build or test, against the declared PATTERN-gate-slot and PATTERN-test-tiers texts, and D8 names no bound for it
    fix: In D8, state what the card Verify runner now lets through unslotted and pick a bound that keeps the declared PATTERN texts true. Either have the plan gate refuse a Verify line that runs a build or test other than through lyx gate test (as D9 does for tree_scope_verify), or keep a single gate slot in the runner for any Verify line that is not a lyx gate test call. Then add an acceptance criterion that pins the chosen rule.
  - id: F2
    severity: BLOCKING
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:75 (D2), :95 (D3), :141 and :148 (D7)
    summary: Under the none scoper the zero-test and floor rules contradict each other. D7:141 makes a run with words after -- not whole, but D7:148 says every none run is whole and the floor applies. D2 passes explicit targets verbatim, but D3 sets none's targets to all_targets on every run. A filtered or explicit-target C# run is therefore both floor-checked and not.
    fix: Write one rule in D7 for the none scoper and match D2 and D3 to it. Recommended rule. Under none, a call naming explicit targets is either refused naming --all, or passed verbatim as a narrower run with no floor. A run with words after -- is never whole and takes no floor. Drop "Under none every run is whole, so nothing is dropped and the floor applies" from D7, or restate it as "computed and card scopes are whole under none". Add an acceptance criterion for a filtered none run and an explicit-target none run.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:164-165 (D8 table), :187 (D9)
    summary: D8 gives darn's Merge-In, catch-up merge-ins, Finalize and Publish the landing list over --all with no tree-scope exception, while D9 keeps tree_scope's meaning and today a darn Merge-In and Finalize run the tree-scoped command for a tree-scoped diff (darnVerifySource, PATTERN-verified-tree), so D8 silently drops that path
    fix: In the D8 table, add to the Merge-In, catch-up merge-ins and Finalize row that a darn run with a tree-scoped diff runs tree_scope_verify instead. State that a darn Publish of a tree-scoped diff skips both the landing and the publish lists, or only publish, whichever keeps today's behaviour. Add the darn tree-scoped landing case to the acceptance criteria.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md:75 (D2 default scope), :187 (D9 tree_scope_verify)
    summary: The hub requires tree_scope_verify to run with an empty {targets} for a docs-only or test-less tree diff, but under D2 a lyx gate test with no targets takes the computed scope, which impactset widens to whole for a non-Go tree, so a tree-scoped diff the hub declared test-free runs every listed tier whole
    fix: In D9, state what a tree-scoped command with an empty {targets} runs. Either darn skips the tree-scoped command when the rendered target list is empty, logging it, or the rendered command carries an explicit scope form whose empty value is reported as skipped with nothing to test. Add the empty-target tree-scoped diff to the acceptance criteria.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:79 (D2 --wait), :174-175 (D8 last bullets), :187 (D9)
    summary: Once the slot moves into the gate, the slot wait of a Verify line or tree-scoped lyx gate test call falls inside the card runner's 10-minute or verifytree's 60-minute timeout, and neither form is required to pass --wait, so on a busy hub such a call exits 75 at the keep-cache-warm cap or times out and fails a correct gate, against D2's claim that a busy hub never fails a gate
    fix: In D9, require a non-empty tree_scope_verify to start with lyx gate test --wait. In D8, have the card runner and verifytree start their timeouts only once the gate reports its slots held, or exempt a lyx gate test command from them as D8 already does for the gate list. In D10, have the plan gate refuse a lyx gate test Verify line that lacks --wait. Add the busy-hub case to the acceptance criteria.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:35 (D1), :98 (D3), :144 (D7), :270 (acceptance)
    summary: D1 says a command without {targets} always runs whole even when a narrower scope is named, while D3 and D7 skip a tier whose computed scope is empty, so for a .md-only diff the record says both that such a tier runs whole and that it is skipped
    fix: In D1, state the order of the two rules. Recommended. A tier whose command lacks {targets} ignores the computed or card scope, the drop rule never applies to it, and an empty computed scope skips it like any other tier. Or say instead that it always runs. Make the acceptance criterion at line 270 name the chosen behaviour.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:101 (D3 Bound), :144 (D7 drop rule), :223 (D12 tier1)
    summary: D3's Bound that the go scoper narrows exactly where impactset narrows today is false. The D7 drop rule removes test-less packages from {targets}, so tier1's go vet {targets} no longer vets a changed package that has no tests, and a fully dropped scope skips the go build ./... that today's command runs even for an empty set.
    fix: Rewrite D3's Bound to name the extra narrowing, test-less changed packages are neither built nor vetted at card, batch and round gates and are caught only by final and landing, or keep them covered by stating in D7 that dropping applies to test targets only, with tier1's build and vet running over the undropped set.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md:57-59 (D1 load), :215 and :227 (D12)
    summary: D1 claims a hub midway through a key-by-key lyx config gate set still loads, but D1 validates each tier's own key floors at load and lyx config refuses any value the loader refuses, so a tier set one key at a time is refused. D12 derives timeout_min and min_tests from a measured whole run, which needs those keys set first, and the record never states the values before measurement or how the open map tiers, with its nested env and covers, is written through --set.
    fix: In D1, state which per-tier keys load validation requires, and drop the key-by-key claim or make missing per-tier keys a use-time refusal like the cross-references. In D12, state the provisional timeout_min and min_tests the orch sets before measuring, and the lyx config gate route (one multi---set call, or the editor) that writes a whole tier.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md:81 (D2 timeout), :113 (D5), :123 (D6)
    summary: D5 runs the guard step through guard_tier's command after the listed tiers pass, but when guard_tier is not in the call's list, as for loomyard's card gate [0] with guard_tier 1, D6 has released every slot and no listed tier's timeout_min covers the step, so its slots, timeout and prebuilt_lyx are undefined
    fix: In D5, state that the guard step is a run of guard_tier. It acquires guard_tier's slots and runs under guard_tier's timeout_min and prebuilt_lyx setting, whether or not that tier is listed. Add it to D11's logged outcomes. Correct D2:81 to match.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md:140 (D7), :172 (D8 confirm-the-fix)
    summary: The round's confirm-the-fix pass reruns each failing test by name, which is a run with words after --, and D7 fails such a run when it discovers zero, so a fix that renames or deletes the failing test fails the round gate on every attempt, where today a go test -run with no match passes
    fix: In D8, state that a confirm-the-fix run discovering zero is reported as skipped with nothing to test, the test no longer existing, rather than failed. Or state that the round then runs the failing tier over the record's packages without the name filter. Add the renamed-test case to the acceptance criteria.
  - id: F11
    severity: MEDIUM
    class: decision
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:146 (D7)
    summary: D7's rule that a package holds tests when a _test.go file declares a Test, Benchmark, Fuzz or Example function keeps packages that discover zero under go test -json, such as a Benchmark-only package, a package with only output-less Examples, or a file excluded by a GOOS or GOARCH constraint on this host, so such a package fails the zero-test rule instead of being skipped
    fix: In D7, define test-holding as declaring a Test or Fuzz function, or an Example with an Output comment, in a _test.go file that the tier's build_tags and the host's GOOS and GOARCH compile. Add a Benchmark-only package to the acceptance criterion at line 272.
  - id: F12
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:128 (D6 Lease.Env), :220 and :223 (D12 tier env and tier1)
    summary: With Lease.Env carrying only the inheritance variable and loomyard's tier env setting GOFLAGS to -trimpath alone, tier1's go build ./... and its go vet runs lose the -p cap they get today, so they run at GOMAXPROCS parallelism inside one slot and defeat the tier's parallel
    fix: In D12, set every loomyard tier's env to GOFLAGS -trimpath -p={parallel}, so every go command in the tier takes the cap. Add the cap on build and vet to the acceptance criteria.
  - id: F13
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md:64 (D1 go_parallel), :128 (D6 Lease.Env), :286 (PATTERN-gate-slot)
    summary: Two existing slot holders outside lyx gate test, planglyph's slotted type load and cmd/testtiming's slotted measured go test, lose the -p cap and -trimpath when Lease.Env shrinks, and testtiming loses its parallelism source when go_parallel retires, while testtiming's slotted go test also contradicts the declared PATTERN-gate-slot and PATTERN-test-tiers texts. The record is silent on both.
    fix: Add a decision for slot holders that are not a gate tier. State the GOFLAGS that planglyph's type load sets itself under its slot. Either route cmd/testtiming's measured run through lyx gate test, with its parallelism taken from the tier, or name it as an exception in the declared PATTERN-gate-slot and PATTERN-test-tiers texts.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:162, :164, :168 (D8), :235 (D12 NORCE gates)
    summary: Where final and landing differ, as in NORCE Models, Merge-In's pass keys on the landing command and verifytree drops every other command's entry naming another tree except the Site's base command, and the record does not say which base command Merge-In names, so Merge-Burler-Fix may find no final pass and fall back to a whole final run
    fix: In D8, state the base command Merge-In and Publish name on Site, recommended the round's base command, loom's final and darn's landing, so the base entry survives a landing-list pass.
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:76 (D2 -C), :97 (D3 per-module runs)
    summary: D2 says -C sets the run directory for explicit targets and overrides the tier's dir, while D3 runs a computed go scope once per module in that module's directory, and the record does not say whether -C with a computed or card scope overrides those directories or is refused
    fix: In D2, state that -C is accepted only with explicit targets and refused with another scope form, naming the explicit-target form, or state what it overrides under a computed scope.
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:35 (D1), :114-115 (D5)
    summary: D5 refuses a guard_tier command lacking {targets} or {args} at use, but also skips the guard step after a whole guard_tier run, and the record does not order the two, so a call that lists guard_tier may be refused for a step it would skip
    fix: In D5, state that the refusal applies only when the guard step runs, after the skip check.
  - id: F17
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:86 (D2 Bound)
    summary: D2's Bound that --base, --card and explicit targets never change which tiers run is broken twice. A card or computed scope adds D5's guard step through guard_tier's command, a tier the call may not list. A scope whose targets are all dropped skips every listed tier (D7).
    fix: Reword D2's Bound to say the scope forms never add a tier to the list, except the guard step D5 runs, and may skip a listed tier with nothing to test under D7.
  - id: F18
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:200 (D10 Bound)
    summary: D10's Bound names only a script or bash -c as what the refuse_raw row lets through, but a prefix match in command position also misses go -C dir test, which the raw-go row covers today, and a command behind an assignment or launcher prefix such as GOFLAGS=x go test or env go test
    fix: In D10's Bound, list the forms the row lets through, a script, bash -c, a flag between the prefix's words such as go -C dir test, and an assignment or launcher prefix, or extend the row's match to cover them.
  - id: F19
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:187 (D9)
    summary: D9's claim that requiring tree_scope_verify to start with lyx gate test makes the tree-scoped command slotted does not hold for a compound value such as lyx gate test ... && dotnet test X, whose tail runs unslotted under verifytree's timeout
    fix: In D9, either refuse a tree_scope_verify holding a shell control operator, or reword the claim to say only that its leading command is slotted, the key being hub-file and operator-owned.
  - id: F20
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:131 (D6 Bound), :258 (Open risks)
    summary: During the deploy transition, processes still running the old binary, such as an in-flight card runner or verifytree, take single slots with no knowledge of the drain lock, so a new slots all waiter can be passed by them until they exit, which D6's Bound and Open risks do not name
    fix: Add to Open risks that the drain does not hold back pre-deploy processes, which keep acquiring single slots until they exit.
  - id: F21
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:57-59 and :63 (D1)
    summary: A machine file's tiers entry holding only parallel for a tier the hub file no longer declares stacks into a tier with only parallel, which D1's per-tier load validation refuses, failing every gate config reader on that machine, slot-only readers included, against D1's claim that they still load
    fix: In D1, state that a machine-layer parallel for an undeclared tier is dropped at load with a logged warning, or that the machine layer may only override tiers the hub file declares and is refused naming lyx config gate --layer machine.
  - id: F22
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:84 (D2)
    summary: D2 says the CPU watchdog applies to each tier command, but the watchdog quits only processes whose path ends in .test, so a vstest tier's test host is never watched and for NORCE Models the claim holds in name only
    fix: In D2, state that the CPU watchdog watches Go test binaries only and a vstest tier is bounded by its timeout_min alone.
  - id: F23
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md:182-184 (D9), :195 (D10)
    summary: The webster fork and recovery prompts still render the plan's verify section for an in-flight plan, which under refuse_raw shows forks raw go test commands their guard now refuses, and D9 does not say the prompts stop rendering it
    fix: In D9, state that the webster fork and recovery prompts stop rendering the plan's verify section and render the hub's tier lines and gate lists instead.
---

The record's tier, slot and site design fits the board entry's operator decisions.
Most decisions state their bounds.
The defects cluster in four places:
the zero-test and floor rule under the `none` scoper,
the paths that run tests or builds outside a tier once `verifytree` and the card runner drop their slots,
the darn tree-scope paths,
and bootstrapping hub values the orch must apply.

### [BLOCKING] Card `**Verify:**` runner runs unslotted builds and tests

**Location:** `_lyx/discussion/decision-record.md:174`, `:198`

**Issue:** D8 says the card `**Verify:**` runner no longer takes a gate slot, and that every command it runs "is a `lyx gate test` call or is bounded as D9 and D10 say".
D10 refuses only a `refuse_raw` prefix, which is `go test` for loomyard, in a `Verify:` line.
A `Verify:` line such as `go build ./...`, `go vet ./...`, `make test` or a script that runs tests therefore runs with no slot.
Today `rerunCardVerifies` slots it (`internal/websterengine/cardverify.go:28-31`).
That is a gate build or test outside a `lyx gate test` tier run, against both declared texts:
`PATTERN-gate-slot` ("only inside a `lyx gate test` tier run") and `PATTERN-test-tiers` ("only `lyx gate test`").
D8 also introduces a downgraded guard without stating what it lets through, which the rubric's attack-surface rule flags.

### [BLOCKING] Zero-test and floor rule contradicts itself under `none`

**Location:** `_lyx/discussion/decision-record.md:75`, `:95`, `:141`, `:148`

**Issue:** D7:141 says a run with words after `--` is not whole, so a filtered run takes no floor.
D7:148 says "Under `none` every run is whole, so nothing is dropped and the floor applies".
Take NORCE Models `lyx gate test --tier 0 -- --filter X`, discovering 3 tests against a floor in the hundreds: line 141 passes it and line 148 fails it.
D2:75 passes explicit targets verbatim as `{targets}`, but D3:95 makes `{targets}` `all_targets` on every `none` run.
So an explicit-target C# run is left as refused, silently widened, or narrow but floor-checked.
The focus directive's first question, one consistent rule for explicit targets and words after `--`, is not met for `none`.
It is met for `go` under `--card` and `--base`.

### [MEDIUM] Darn landing sites lose the tree-scoped path

**Location:** `_lyx/discussion/decision-record.md:164-165`, `:187`

**Issue:** `pattern/PATTERN-verified-tree.md` and `darnVerifySource` (`internal/loomcli/darnwiring.go:53-71`) run the tree-scoped command at darn's Merge-In and Finalize for a tree-scoped diff, and skip `publish_verify` at Publish.
D8's table gives the tree-scope exception only to Darn-Write and Darn-Burler-Fix, while D9 claims `tree_scope` keeps its meaning.

### [MEDIUM] Empty tree-scoped targets widen to a computed scope

**Location:** `_lyx/discussion/decision-record.md:75`, `:187`

**Issue:** The hub's `darn.yaml` requires that `tree_scope_verify` "must run with an empty {targets}, since a docs-only or empty diff expands it so".
Today an empty target list renders `lyx gate test --tags integration ` with no packages, which the gate refuses.
Under D2 the same render takes the computed scope.
impactset falls back to whole for a changed file outside a Go package, so the diff the hub declared test-free runs whole.

### [MEDIUM] Slot wait inside the caller's timeout, no `--wait` required

**Location:** `_lyx/discussion/decision-record.md:79`, `:174-175`, `:187`

**Issue:** Today `rerunCardVerifies` and `verifytree.Verify` start their timeouts once their own slot is held (`internal/websterengine/cardverify.go:28`, `internal/verifytree/doc.go`).
After D8 the wait happens inside the `lyx gate test` child, so it counts against the 10-minute or 60-minute timeout.
Without `--wait`, the call exits `SlotBusyExit` at the keep-cache-warm cap.
D9 requires only the `lyx gate test` prefix, and no rule requires `--wait` in a `Verify:` line, so D2's "a busy hub never fails a gate" does not hold for these two paths.

### [MEDIUM] `{targets}`-less tier: always whole, or skipped?

**Location:** `_lyx/discussion/decision-record.md:35`, `:98`, `:144`, `:270`

**Issue:** For a `.md`-only diff, D3 skips every tier, while D1 says a command without `{targets}` always runs whole even when a narrower scope is named.
The focus directive's question about a tier lacking `{targets}` is left open.

### [MEDIUM] D3's Bound is false under the drop rule

**Location:** `_lyx/discussion/decision-record.md:101`, `:144`, `:223`

**Issue:** `internal/impactset/doc.go` says today's command vets "over the set", and that "An empty set runs only the build and the guard step".
Under D7's drop rule, a test-less changed package leaves tier1's `go vet {targets}`.
A fully dropped scope skips tier1's `go build ./...` as well.
The narrowing goes past impactset's, and D3 does not state it.

### [MEDIUM] Key-by-key set and pre-measurement values

**Location:** `_lyx/discussion/decision-record.md:57-59`, `:215`, `:227`

**Issue:** `lyx config gate --help` says "A value the module's loader refuses is refused, naming the key", and that `--set` writes an open-map entry "with a scalar value".
A tier written with `--set tiers.tier0.command=...` alone fails D1's per-tier floors, so the claimed key-by-key path fails.
D12's measured `timeout_min` and `min_tests` need a loadable tier before they exist.
On the focus directive's D12 question: Open risks bounds the deploy-to-set window by refusal and re-step.
What runs before measured values exist is not stated.

### [MEDIUM] Guard step without an owning tier

**Location:** `_lyx/discussion/decision-record.md:81`, `:113`, `:123`

**Issue:** loomyard's `card: [0]` with `guard_tier: 1` runs tier1's command after tier0 has released its slot.
D2:81 counts the guard step in "a tier's" timeout, but no listed tier owns it.

### [MEDIUM] Confirm-the-fix fails on a renamed test

**Location:** `_lyx/discussion/decision-record.md:140`, `:172`

**Issue:** `publishFailureSteps` (`internal/loomshed/publishfailure.go:84-104`) reruns each failing test with `-run '^Name$'`.
Under D7 the gate's equivalent is a filtered run, and a filtered run that discovers zero fails.
A fix that renames or removes the failing test therefore cannot pass the round gate.

### [MEDIUM] Test-holding definition admits zero-discovery packages

**Location:** `_lyx/discussion/decision-record.md:146`

**Issue:** `go test` runs a Benchmark only under `-bench`, and an Example only when it carries an output comment.
A file excluded by a GOOS or GOARCH constraint compiles to nothing on that host.
Each case is kept by D7's rule, then discovers zero and fails the tier.

### [MEDIUM] Build and vet lose the `-p` cap

**Location:** `_lyx/discussion/decision-record.md:128`, `:220`, `:223`

**Issue:** `gateslot` puts `-p=<GoParallel>` in GOFLAGS for every go command in a slot today (`internal/gateslot/doc.go`, Inheritance).
D6 removes that, and D12's tier `env` sets only `-trimpath`, so tier1's `go build` and `go vet` run uncapped.

### [MEDIUM] Slot holders outside the gate

**Location:** `_lyx/discussion/decision-record.md:64`, `:128`, `:286`

**Issue:** `internal/planglyph/typesload.go:56-67` calls `lease.Env(env)` for its slotted type load.
`cmd/testtiming/resources.go:120-149` returns `lease.Env(baseEnv)` and `limits.GoParallel` for its measured `go test`.
Both lose their GOFLAGS under D6, and testtiming loses its parallelism with `go_parallel`.
testtiming's run is a test outside `lyx gate test`, against the declared `PATTERN-test-tiers` text.

### [LOW] Merge-In's base command under unequal lists

**Location:** `_lyx/discussion/decision-record.md:162`, `:164`, `:168`, `:235`

**Issue:** `internal/verifytree/doc.go` says a pass drops other commands' entries naming another tree, except the base command named on Site.
D8 never names Merge-In's base command when `final` ≠ `landing`.
On the focus directive's D8 question: the shared verified-tree entry holds for loomyard's equal lists, and the `--base` fallback to the base command holds when no pass is recorded.

### [LOW] `-C` with a computed scope

**Location:** `_lyx/discussion/decision-record.md:76`, `:97`

**Issue:** D2 does not say whether `-C` with a computed or card scope overrides D3's per-module directories or is refused.

### [LOW] Guard-tier refusal ordering

**Location:** `_lyx/discussion/decision-record.md:35`, `:114-115`

**Issue:** D5's at-use refusal and its whole-run skip are not ordered.

### [LOW] D2's Bound on which tiers run

**Location:** `_lyx/discussion/decision-record.md:86`

**Issue:** D5's guard step adds a run of an unlisted tier's command, and D7's drop rule skips listed tiers, so the Bound's "never which tiers run" is broken twice.

### [LOW] D10's Bound is incomplete

**Location:** `_lyx/discussion/decision-record.md:200`

**Issue:** `internal/shellguard/rows.go`'s `goSubcommandPattern` already covers `go -C <dir> test` for the `raw-go` row.
A plain prefix match does not cover it, nor an assignment or launcher prefix.

### [LOW] Prefix check does not slot a compound command

**Location:** `_lyx/discussion/decision-record.md:187`

**Issue:** A compound value passes D9's prefix check, and its tail runs unslotted.

### [LOW] Drain across the deploy transition

**Location:** `_lyx/discussion/decision-record.md:131`, `:258`

**Issue:** Old-binary processes acquire single slots without reading the drain lock.

### [LOW] Stale machine-layer tier entry

**Location:** `_lyx/discussion/decision-record.md:57-59`, `:63`

**Issue:** On the focus directive's D1 question: the decision does bound the machine override, to integer `slots` and `parallel`, neither of which reaches shell text.
The one gap is a stale per-tier `parallel` that outlives the hub's tier and then fails D1's per-tier load validation.

### [LOW] CPU watchdog claim for `vstest`

**Location:** `_lyx/discussion/decision-record.md:84`

**Issue:** `internal/gatecli/doc.go` limits the watchdog to processes whose path ends in `.test`.

### [LOW] Webster prompts still render `## verify:`

**Location:** `_lyx/discussion/decision-record.md:182-184`, `:195`

**Issue:** `internal/websterengine/beginbatch.go:653` and `recoverbatch.go:311` render `deps.Plan.Verify` into fork prompts.

## Rejected

- Relocation of the Scope "Out" bullet's reason about Describe (line 24) and D11's 87-minute rationale (line 211): each is a one-clause reason the plan writer uses to keep the item out of scope or to place the log, so neither is deliberation narrative to move.
  On the focus directive's relocation question, no Decision passage needs relocating.
- `slots: all`, the drain lock, the inherited-slot rule and `--wait` as unbounded escape hatches: D6:131-132 and D2:86-87 state what each lets through and what bounds it, and the bound-breaker lens's deadlock trace held.
