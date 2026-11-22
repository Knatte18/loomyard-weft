---
verdict: BLOCKING
findings:
  - id: F12
    severity: BLOCKING
    class: consistency
    location: internal/hardenshed/campaign_rows.go:24
    summary: "The new focusFileFormat constant repeats the literal round-%d-focus.md that shedadapters.focusPath (internal/shedadapters/round.go:120) already owns, so the charter path RoundFacts names is a second declaration of one stale-able name, against PATTERN-code-conventions' Named constants rule."
    fix: "Delete focusFileFormat from internal/hardenshed/campaign_rows.go. In internal/shedadapters/round.go export the focus-path builder (rename focusPath to FocusPath, updating every caller in internal/shedadapters, its tests included), and have Campaign.RoundFacts call shedadapters.FocusPath(runDir, round) for the Charter fact; hardenshed already imports shedadapters. Leave the literal in TestCampaignRoundFactsNameTheCharterAndTheOperatorsAsk, which pins the name independently."
  - id: F13
    severity: BLOCKING
    class: design
    location: internal/gateslot/template.yaml:44
    summary: "Retiring gates.harden-live-copies and running the copies under gates.harden-live changes what a hub's gate.yaml values mean, and the diff ships no configreg.Reinterpretation, against PATTERN-config-layers' Config reinterpretation rule. This hub's own gate.yaml sets harden-live to tiers [0, 1] and harden-live-copies to [1]; after the diff CheckCampaign refuses every non-LLM campaign with packages and concurrency above 1 (two tiers when copies run), and the copies key the operator set is silently ignored. The landing summary still says the copies key is neither folded nor derived, names three harden keys, and calls the door two-way with blast radius hardener, although the diff changes what a config setting means."
    fix: "Add a Reinterpretation to the gate Module in internal/configreg/configreg.go (Reinterpretations field), its Rewrite implemented in internal/gateslot beside the template. For a hub base file whose gates map holds harden-live-copies: remove the entry when harden-live then names exactly one tier below tier3 and it equals harden-live-copies; otherwise leave harden-live unchanged and return a flagged line naming the way forward, which is that the copies now run under gates.harden-live, so set harden-live to one tier with `lym config gate --set 'gates.harden-live={tiers: [<live tier>], scope: diff}'` from the prime, then remove harden-live-copies with lym config reconcile. Add a table-driven test that reaches the rewrite and asserts the removal case, the flag text, and a file without the key left unchanged. Then update _lym/landing/summary.md: the Design decisions bullet on the copies key (it is folded into harden-live), the No hub gate writes bullet (two harden keys), and the Merge danger section's door and blast radius so they name the gate.yaml meaning change."
prior:
  - id: F1
    status: resolved
    answer: "RoundFacts opens with the round's focus file and the decision record path, the rubric's judge section makes those two files the judge's one reading exception, and the test covers round 1 (the seed's charter) and round 2."
  - id: F2
    status: resolved
    answer: "TestHardenCampaign_EntryRefusesAnEntryTheHubsBoardLacks drives the production Entry seam over a hub board and asserts the slug, hardener.packages and the resume clause; TestReadBoardEntry covers the found entry and both read failures."
  - id: F3
    status: resolved
    answer: "hardenPreflight reads the entry when the config names no package, so a missing entry is refused before any agent spends tokens; a seed-time SeedBrief failure is left a transient read error on the Bouncer's existing degrade path, as the preferred fix allowed."
  - id: F4
    status: resolved
    answer: "The copies call runs under harden-live and CheckCampaign applies the one-tier check to it, the board entry's folding option. The missing hub-config migration that change needs is the new F13."
  - id: F5
    status: resolved
    answer: "The rubric now carries the merge bar, the safety pass's no-invented-work rule and a count the focus never states; the seed keeps a fan test off the ban when the focus is that path; the suite, determinism and overview/PATTERN rules reach the fixer."
  - id: F6
    status: resolved
    answer: "composeFixPrompt now appends the profile's target instructions to the work statement, and the HardenFix row carries the fix-half rules there. Commits follow burler's source-scope rule, which commits each finding alone unless findings share a file."
  - id: F7
    status: resolved
    answer: "TestHardenRoundKey's BouncerRowSeedsFromTheNamedStencilFilledWithTheCampaignsBrief builds the row with seed_stencil and harden_round, drives the seed through the fake shuttle and asserts the prompt holds the campaign's brief."
  - id: F8
    status: resolved
    answer: "hardenPreflight returns gateslot.LoadConfig's ErrConfigAbsent error, which names lym fabric reconcile, and TestHardenPreflight's AHubWithNoGateConfigIsRefused reaches it."
  - id: F9
    status: resolved
    answer: "TestHardenPreflight writes a sparse hub gate.yaml with no harden key, so the passing rows read the template's harden-live."
  - id: F10
    status: resolved
    answer: "Commit 8277b4ab carries Loosened-Check trailers naming each removed TestCheckCampaign row and TestHubSetsGate with the retiring design."
  - id: F11
    status: resolved
    answer: "The default new subtest asserts exactly two bares in the hub's container, beside the single-repo subtest's one bare holding refs/lym/."
---

### [BLOCKING] RoundFacts redeclares the Bouncer's focus filename

**Location:** internal/hardenshed/campaign_rows.go:24

**Issue:** `focusFileFormat = "round-%d-focus.md"` was added by the F1 fix.
The Bouncer's seed output, judge outputs, focus reader and resume probe all build that path through `focusPath` in `internal/shedadapters/round.go:120`.
The two agree today, but a rename of the owning builder would leave the judge's Charter fact naming a file that no longer exists, with no test failing: the RoundFacts test pins the hardenshed copy, not the Bouncer's.
`PATTERN-code-conventions` requires a name that can go stale to be one constant declared once, and the constraints section makes every listed-PATTERN violation BLOCKING.
The assistant raised the same defect; it is merged here.

### [BLOCKING] The harden-live-copies retirement ships no config reinterpretation

**Location:** internal/gateslot/template.yaml:44

**Issue:** The F4 fix removes `harden-live-copies` from the template and from `goReadGates`, and `NewLiveGate` now runs the copies under `harden-live`.
`CheckCampaign` now requires `harden-live` to name one tier whenever copies run.
Before this change, `harden-live` with several tiers beside a one-tier `harden-live-copies` was the valid shape the template comment asked hubs to set.
The hub this run belongs to holds exactly that shape (`_bolt/_lym/config/gate.yaml`: `harden-live: tiers [0, 1]`, `harden-live-copies: tiers [1]`).
With this diff its next hardener campaign that names packages, at the template's concurrency and with no LLM driving, is refused at Preflight with "gates.harden-live names the tiers [0 1], and --copies runs one tier".
The copies tier the operator chose is dropped without notice.
`PATTERN-config-layers` says a change to what a hub config value means ships a `configreg.Reinterpretation`, which `hubreconcile` applies once per hub to rewrite the value or flag it with a way forward.
No reinterpretation is declared; `grep Reinterpretation{` finds none in production code.
The landing summary contradicts the diff on the same point.
Its Design decisions say "The copies key is not folded or derived", and that the template "already gives `harden-live-copies` one tier".
Its No hub gate writes bullet names "all three harden keys".
Its two-way door with blast radius `hardener` is the rubric's Merge danger case: a diff that changes what a config setting means.

## Focus verdicts

- Judge inputs: RoundFacts names round N's focus file and the decision record; round 1's focus file is the seed's charter, so round 1 is covered. The filename's ownership is F12.
- Missing-entry refusal: reached by `TestHardenCampaign_EntryRefusesAnEntryTheHubsBoardLacks` with the slug, `hardener.packages` and `lym loom resume` asserted. A seed-time SeedBrief failure still degrades through `runSeedSpawn`'s log-and-return to the empty-lists fallback (`internal/shedadapters/bouncer.go:1811-1814`). Preflight now refuses the missing entry first, so only a transient board read between Preflight and the seed reaches that path; no finding.
- Copies tier: copies run under `harden-live`, and `hardenPreflight` refuses a hub with no gate.yaml through `LoadConfig`'s reconcile way forward before any agent runs. The config migration the fold needs is F13.
- Fix-half rules: they reach the fixer through `profile.target.instructions`, which `composeFixPrompt` appends to the work statement. The rubric and the seed carry the Crucible template's and README's rules, checked against `git show 7346d6a8^:crucible/review-prompt-template.md` and `README.md`. Burler's fix step already requires fixing every finding, NIT included.
- Seed brief wiring: `TestHardenRoundKey` asserts a `harden_round` row's seed prompt carries the campaign's brief through the named `seed_stencil`.

I ran no gate: the verify gate's reports match HEAD's tree, and the assistant's tier-0 attempt could not take a slot in this sandbox.

## Rejected

- Assistant's carried F6 (one commit per finding, overriding burler's shared-file grouping for HardenFix): Hardener's fix prompt already carries a commit-per-finding rule through burler's source-scope rules, and its shared-file grouping keeps each commit green and every finding committed. A second, competing commit rule for one recipe is not what the Crucible carry-over asks for. The fixer's fixed-differently reason holds.
