---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/hardenshed/campaign_rows.go#Campaign.RoundFacts
    summary: "The rubric's judge rules (carry the charter forward, re-seed from it, never steer outside the operator's focus) cannot be followed, because the judge's inputs name neither the current round's focus file (the charter) nor the decision record. Round 2's focus file therefore loses the charter: what to read, the layouts, the cost declaration's execution ban and the teardown check."
    fix: "Give the Harden-Bouncer judge the charter and the operator's ask through its facts. In Campaign.RoundFacts (internal/hardenshed/campaign_rows.go), append a fact line naming the absolute path of this round's focus file (round-<round>-focus.md in runDir) as the charter to read and restate, and the decision record's path as the operator's ask whose stated focus binds the next focus file. Do not change the shared bouncer-template-judge.md, so other recipes are unchanged. Add a row to the RoundFacts test asserting both paths appear in the facts. Also cover the round-1 seed charter path: the judge of round N reads round-N-focus.md, which for N=1 is the seed's charter."
    origin: lens:code-scope
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/loomcli/hardenwiring.go:179
    summary: "The new refusal 'the board holds no entry ...; way forward: name the modules ... hardener.packages' in a bound module (internal/loom*) has no test that reaches it and asserts its message, against PATTERN-refusal-way-forward. The seam's present-entry path and its open-board and GetTask error paths are untested too."
    fix: "Add a tier0 table-driven test in internal/loomcli (for example hardenwiring_test.go) that builds hardenCampaign over a hub fixture and calls its Entry seam. One row has a board holding the run's entry and asserts the returned title and labels. One row has a board without it and asserts the error text holds the slug, hardener.packages and lym loom resume. If F3's fix moves the refusal, the test asserts it where it then surfaces."
    origin: lens:code-tests
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/shedadapters/bouncer.go:1925
    summary: "A SeedBrief failure, the missing-board-entry refusal included, is swallowed. runSeedSpawn only logs the render error and returns nil (bouncer.go:1811-1814), and seedCall then writes the empty-lists fallback through ensureFocus(1) and returns Stuck. Round 1 runs with no charter, the way forward reaches no one, and the parsed fallback stops a later re-seed. The landing summary's claim that a missing entry fails with a way forward naming hardener.packages is not borne out."
    fix: "Surface a seed-brief failure as a refusal instead of a degraded seed. Preferred: have hardenPreflight call the campaign's SeedBrief (or Entry when cfg.Packages is empty) and return its error, so Preflight refuses before any agent spends tokens, with the existing way forward. Then a seed-time failure is a transient read error only. Alternatively, in seedCall, return Stuck with the brief error's text as the pointer Reason and skip ensureFocus(1) when the render error came from SeedBrief. Add the reaching test that F2 asks for at whichever site carries the refusal."
    origin: lens:code-correctness
  - id: F4
    severity: MEDIUM
    class: scope
    location: internal/hardenshed/live.go:180
    summary: "The board entry asks that the copies run derive its single tier or be folded into harden-live with a copies count. The diff does neither: harden-live-copies stays an independent key at tier 0. A hub that raises harden-live to its live tier therefore runs copies on tier 0 unless it also writes the copies key. The summary's premise that neither option fits without changing gate's refusals is false, because gatecli accepts --gate harden-live --copies N with targets whenever harden-live names one tier below tier3 (internal/gatecli/run.go:400 and :450-453)."
    fix: "Derive the copies run's tier from harden-live. In NewLiveGate, run the copies call under LiveGateKey (deps.Run(LiveGateKey, deps.Concurrency, deps.Packages)) unless the hub's own gate config sets harden-live-copies, which stays the override. In CheckCampaign, apply the one-tier check to whichever key the copies run uses. Update the gateslot template.yaml comment, the hardenshed doc's Live gate section and the NewLiveGate doc. Add a TestLiveGate row and a TestCheckCampaign row for a hub whose harden-live is tier 2 with no copies key set, asserting the copies call uses harden-live."
    origin: lens:code-scope
  - id: F5
    severity: MEDIUM
    class: scope
    location: contracts/stencils/harden/harden-rubric-review.md:83
    summary: "The added entry 'Carry over every rule of the template and README that Hardener's rubric and stencils lack' is not fully met. Still missing: the merge-bar calibration (a timeout under the N-copies stress peg is no defect); the safety pass's 'no new defects is the expected outcome, do not invent work'; the fan-test exception to the execution ban; extending a maintained tools/sandbox SANDBOX-<MODULE>-SUITE.md and keeping its coverage guard green; proving a new live test's determinism by running it many times in parallel under load; updating docs/overview.md and PATTERN.md when invariants move. Contradicted: the pre-count is kept where the round never sees it, with its blind spots recorded, but 'Explain any difference from a count the focus states' shows the count to the round."
    fix: "In harden-rubric-review.md, add or amend bullets. Under Rules of a round: the merge bar (a timeout under the concurrent copies' CPU peg is a diagnostic, not a defect); determinism proof for a new live test; extend a maintained tools/sandbox suite file when a round surfaces behaviour it lacks, keeping cmd/lym's sandbox coverage test green, with no new suite required when none exists; the fix updates docs/overview.md and PATTERN.md too when invariants or the module table move. Under the safety-pass item: no new defects is the expected outcome, and the round must not invent work. Replace the Count bullet's 'a count the focus states' with the round stating its own count, the enumeration command and what the command cannot see, so the focus states no count. In harden-template-seed.md's cost declaration, keep a fan test off the execution ban when the operator's focus is that fan or cluster path, and keep the rubric's ban bullet consistent with that."
    origin: lens:code-scope
  - id: F6
    severity: MEDIUM
    class: design
    location: contracts/stencils/harden/harden-rubric-review.md:100
    summary: "The diff puts new fixer obligations into the review rubric: fix every finding, one commit per finding; defer large fixes with a reason; leave a live regression test that updates the package doc. The HardenFix fixer never receives the rubric: burlerengine's fixer prompt carries no rubric, fasit, focus file or prior rounds (internal/burlerengine/doc.go:85). So these rules reach the reviewer, who fixes nothing, and never the fixer."
    fix: "Deliver Hardener's fix-half rules to the HardenFix row's fixer. Add a hardener fix instruction stencil, or a profile instruction on the HardenFix row of hardener-recipe.yaml that the burler fix prompt already renders, carrying: fix every finding, NIT included, one commit per finding; defer a large fix with its reason; sabotage-prove each new regression test; redeploy through lym harden sandbox new; leave a tier-2 or tier-3 live regression test that waits on a deadline; update the package doc in the same commit. Leave the review-half rules in the rubric. Other recipes' fixers stay unchanged."
    origin: assist:assist
  - id: F7
    severity: MEDIUM
    class: design
    location: internal/shedrecipe/entries_bouncer.go:368
    summary: "No test checks that a harden_round Bouncer carries the campaign's SeedBrief and its seed_stencil key into the seed prompt (the mutation survivor at :368). The seed_stencil field assignment at :331 is likewise unasserted. Dropping either line leaves every test green while the shipped hardener seed fails to fill seed_brief, or silently reverts to the shared seed template."
    fix: "Add a row to the harden_round tests in internal/shedrecipe/entries_harden_test.go. It builds the Bouncer row with harden_round true and seed_stencil naming a stub stencil written as '{{.seed_brief}} {{.focus_path}}', with an Env.Harden whose campaign names a package. It drives the seed call through the fake shuttle and asserts the captured prompt holds the campaign's brief text."
    origin: lens:code-tests
  - id: F8
    severity: MEDIUM
    class: design
    location: internal/loomcli/hardenwiring.go:298
    summary: "With no hub gate.yaml, hardenPreflight tolerates ErrConfigAbsent and passes empty LiveKeys, so CheckCampaign now passes. Before the diff the HubSetsGate check refused at Preflight. The run now spends the seed and a round's review and fix before the live gate's lym gate test --gate harden-live is refused for the absent config, against CheckCampaign's contract that its refusals come before any agent spends tokens."
    fix: "In hardenPreflight, return the gateslot.LoadConfig error when it is ErrConfigAbsent rather than tolerating it, since the error already carries its way forward (run lym fabric reconcile). Add a TestHardenPreflight row for a hub with no gate.yaml asserting that refusal. Seed a sparse gate.yaml in the passing rows' fixture (see F9)."
    origin: lens:code-correctness
  - id: F9
    severity: MEDIUM
    class: consistency
    location: internal/loomcli/hardenwiring_test.go:119
    summary: "TestHardenPreflight's doc says the hub passes 'with the template's values', but its hubLocation fixture writes no gate.yaml. gateslot.LoadConfig returns ErrConfigAbsent with a zero Config, and the row passes without ever reading the template's harden gate keys. So it does not pin the done-when case, a hub whose sparse gate.yaml sets no harden key, and a template change putting harden-live on the billed tier would still pass."
    fix: "In TestHardenPreflight, write a sparse hub gate.yaml into the fixture's board dir (the base keys plus a gates map with no harden key) before calling hardenPreflight, so gateslot.LoadConfig fills harden-live and harden-live-copies from the template. Keep the TheTemplateWithNoPackagePasses and package-missing rows."
    origin: lens:code-tests
  - id: F10
    severity: LOW
    class: consistency
    location: internal/hardenshed/campaign_test.go:49
    summary: "The diff removes test assertions that exist at the review base: the NoPackage, LiveKeyNotSetByTheHub and CopiesKeyNotSet rows of TestCheckCampaign, and TestHubSetsGate. No commit of the range carries the Loosened-Check trailer the rubric requires for a removed check."
    fix: "Add a commit, an empty one with git commit --allow-empty if nothing else lands, whose message carries a Loosened-Check trailer. It names TestCheckCampaign's removed NoPackage, LiveKeyNotSetByTheHub, CopiesKeyNotSetByTheHubWhenCopiesRun, CopiesKeyNotSetWhenTheTestsDriveAnLLM and CopiesKeyNotSetWhenConcurrencyIsOne rows and TestHubSetsGate, and its cause: the board entry's No hub gate writes and the charter written from the title and labels retire those refusals, and the NoPackagePassesAndRunsNoCopiesOrTierTestCheck and TemplateLiveKeysPass rows assert the new behaviour."
    origin: assist:assist
  - id: F11
    severity: LOW
    class: design
    location: internal/hardencli/cli_integration_test.go:79
    summary: "The default 'new' subtest of TestSandboxVerbs asserts only single_repo false, which echoes the flag, and never the hub's layout. A slip that wired the single-repo Seed and Clone into the default would pass, although the default must stay the two-bare hub."
    fix: "In the default new subtest, glob the hub container's *-bare entries and assert exactly two, the warp and weft bares, mirroring the single-repo subtest's one-bare check."
    origin: lens:code-tests
---

Reviewed `git diff 3a9f8d8419fbcd415992ca15a8c6e9d25822015e..HEAD` against the amended board entry and its three added entries (Crucible fill, seed-pass placement, operator-named focus).
The single-repo sandbox, the `seed_stencil` key and the Preflight relaxation are sound.
The judge cannot carry the charter forward, a new refusal is untested, and the seed-brief failure path degrades silently.

### [BLOCKING] The judge never reads the charter it must carry forward

**Location:** `internal/hardenshed/campaign_rows.go#Campaign.RoundFacts`

**Issue:** The rubric's judge section tells each focus file to restate the charter's standing sections and never steer outside the operator's focus.
The shared `bouncer-template-judge.md` reads only the facts file, the latest review and the previous ledger, "and never the artifacts".
`RenderJudgePrompt`'s values (`internal/shedadapters/bouncer.go:2016-2031`) name no current-round focus file and no decision record.
`Campaign.RoundFacts` writes models, live report, strays, commits and deferred findings only.
A judge following its reading order therefore cannot restate what to read, the layouts, the cost declaration's execution ban or the teardown check.
It also cannot see the operator's focus.
Round 2 then runs without the cost declaration's bans, which the rubric itself ties to host memory exhaustion.
The added entry asked for the judge's own stencil text "where the shared Bouncer seed stencil cannot carry this".
The summary's "The judge keeps the shared template" rests on the rubric alone reaching the judge, and the rubric is no input channel.

### [BLOCKING] The missing-board-entry refusal has no reaching test

**Location:** `internal/loomcli/hardenwiring.go:179`

**Issue:** `PATTERN-refusal-way-forward` binds `internal/loom*`: a new refusal lands with a test that reaches it and asserts its message.
`git grep "board holds no entry"` finds this site and no test.
The reach report lists `hardenCampaign` as reached by no mapped test.
`TestCampaignSeedBrief` passes a fake `Entry` closure, so it never reaches the production seam.

### [MEDIUM] A seed-brief failure seeds an empty charter silently

**Location:** `internal/shedadapters/bouncer.go:1925`

**Issue:** `RenderSeedPrompt` returns a `promptRenderError` for a `SeedBrief` error.
`runSeedSpawn` logs it with `logger.Warn` and returns nil (`bouncer.go:1811-1814`).
`seedCall` then runs `ensureFocus(1)`, which writes the empty-lists fallback, and returns Stuck, and the ring proceeds to round 1.
A title-only campaign whose board entry is missing therefore reviews with no module named and no charter.
The way forward naming `hardener.packages` reaches only a log line.
The landing summary's "An entry the board lacks fails the seed brief with a way forward" is a claim the code does not bear out.

### [MEDIUM] The copies tier is neither derived nor folded

**Location:** `internal/hardenshed/live.go:180`

**Issue:** The entry's "No hub gate writes" design asks that the copies run derive its single tier or fold into `harden-live` with a copies count.
`harden-live-copies` stays its own key at template tier 0 (`internal/gateslot/template.yaml:26`), so a hub that raises `harden-live` runs copies on tier 0 unless it writes the copies key too.
The summary says neither option fits because `--gate` refuses `--tier`.
The derivation needs no `--tier`, though: `gateKeyRefusal` (`internal/gatecli/run.go:450-453`) admits `--copies` on any one-tier key below tier3, `harden-live` included, given targets (`run.go:400`).
No `Added after Discussion` entry records the departure.

### [MEDIUM] Crucible's carry-over is incomplete and contradicted in one rule

**Location:** `contracts/stencils/harden/harden-rubric-review.md:83`

**Issue:** Read against `git show 7346d6a8^:crucible/README.md` and `review-prompt-template.md`:
- missing is the merge-bar calibration, in README "Reading the result" and template "State the **merge bar**";
- missing is the safety pass's "do not invent work" (template, Round context);
- missing is the fan-test exception to the execution ban, "Omit a test from the list only when this round's mission is to test that fan/cluster path";
- missing is the maintained-suite rule (README "Grow whatever scenario record you keep"; template Fixing), and `tools/sandbox/` holds such suites, `SANDBOX-FABRIC-SUITE.md` among them;
- missing is the determinism proof under parallel load (template Fixing);
- missing is the `docs/overview.md` and `PATTERN.md` update (template Fixing);
- contradicted is README refinement 1, which keeps a pre-count where the round never sees it and records its blind spots, while the rubric's "a count the focus states" hands the count to the round.

The rest of the template's live-driving rules land:
- live driving yourself;
- the focus list as a floor, with the improvised scenarios named;
- log as you go;
- no cost excuse;
- the existing tests as the fix gate only;
- the cost declaration;
- the teardown check.

### [MEDIUM] The fixer's new obligations sit in a rubric the fixer never reads

**Location:** `contracts/stencils/harden/harden-rubric-review.md:100`

**Issue:** The burler fixer's prompt carries no rubric, fasit or focus file (`internal/burlerengine/doc.go:85`).
The added rules "Fix every finding ... one commit per finding", the size-based deferral and "Leave a live regression test ... updates the module's package doc" govern the fix half.
Written only into the rubric, they reach the reviewer, who fixes nothing.

### [MEDIUM] The recipe-level seed wiring is untested

**Location:** `internal/shedrecipe/entries_bouncer.go:368`

**Issue:** The mutation survivor at :368 has candidate killers that all pass with the line dropped.
`TestBouncer_RenderSeedPrompt_NamedSeedStencil` sets `SeedStencil` and `SeedBrief` on the adapter directly.
`TestHardenerRecipe` checks only the YAML value.
The `harden_round` tests in `entries_harden_test.go` exercise facts, live-red and halt, never a seed call.
No test therefore connects the constructor's `seed_stencil` and `SeedBrief` wiring to the seed prompt.

### [MEDIUM] Preflight no longer refuses a hub with no gate.yaml

**Location:** `internal/loomcli/hardenwiring.go:298`

**Issue:** `gateslot.LoadConfig` returns `Config{}` with `ErrConfigAbsent` for an absent hub file (`internal/gateslot/config.go:227-232`).
`hardenPreflight` tolerates that, so `hardenLiveKey` returns empty keys and `CheckCampaign` passes.
The removed `HubSetsGate` check used to refuse this case at Preflight.
The live gate's call is refused only after the seed, review and fix have run (`internal/hardenshed/live.go:176-178`).

### [MEDIUM] TestHardenPreflight never reads the template's gate keys

**Location:** `internal/loomcli/hardenwiring_test.go:119`

**Issue:** `hubLocation` (`internal/loomcli/hubfixture_helper_test.go:76-94`) seeds loom, tapestry and landing config but no gate.yaml.
The "template's values" row therefore passes on the absent-config path with empty keys, not on the template-filled path a real hub's sparse gate.yaml takes.

### [LOW] Removed checks carry no Loosened-Check trailer

**Location:** `internal/hardenshed/campaign_test.go:49`

**Issue:** The rubric counts a removed assertion of a test that exists at the review base as a finding, unless a `Loosened-Check:` trailer names it.
The removals here follow the entry's design, but no commit in the range carries the trailer.

### [LOW] The default sandbox layout is unasserted

**Location:** `internal/hardencli/cli_integration_test.go:79`

**Issue:** The default subtest's only layout signal is `single_repo == false`, which `newVerb` copies from the flag variable.
Nothing checks that the default hub still has two bares.

## Rejected

- Stale fixture text in `internal/preflightshed/preflight_test.go:172`: an arbitrary injected error used to test propagation, which no code asserts as a production message.
- An empty `seed_stencil` failing at construction: the field's contract defines empty as the shared default, and other recipes rely on that. The focus question on it is answered "not a defect".
- `CloneAndWireFromLocalOrigin`, `newVerb`, `sandboxDeps` and `hardenCampaign` listed as reached by no mapped test: `TestSandboxVerbs` drives the first three in-process through `clihelp.ExecuteIn`, and `hardenCampaign`'s gap is F2.
- A config-reinterpretation miss for `packages: []`: the old meaning halted every run at Preflight, so no hub's working value changes behaviour.
- The orchestrator-only "re-drive every BLOCKING fix in its strongest mode": the entry asks for the template's and README's rules, and this one sits in `orchestrator-prompt.md` alone.
- `markers_test.go`'s new row and `TheShippedHardenSeedFills`: they pin a marker set and Go-filled values, which PATTERN-test-speed permits, not static prose.
