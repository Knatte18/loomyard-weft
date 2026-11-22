---
verdict: BLOCKING
findings:
  - id: F12
    severity: BLOCKING
    class: consistency
    location: internal/hardenshed/campaign_rows.go:24
    summary: "The new focusFileFormat duplicates the Bouncer's production focus filename, violating PATTERN-code-conventions' single-declaration rule."
    fix: "Remove focusFileFormat from internal/hardenshed/campaign_rows.go. Export the owning focus-path builder in internal/shedadapters/round.go, update its existing callers, and call that builder from Campaign.RoundFacts so the seed writer, judge outputs, focus reader and charter facts share one filename declaration."
    origin: lens:code-affected-set
  - id: F6
    severity: MEDIUM
    class: design
    location: contracts/recipes/hardener-recipe.yaml:133
    summary: "Hardener's fixer receives its new obligations but still groups findings sharing a file into one commit, contrary to the required Crucible per-finding sequence."
    fix: "Make the HardenFix profile's target instructions explicitly require one green commit per finding before starting the next, including findings sharing a file, and explicitly override the shared source-scope grouping exception for this row. Keep other recipes' commit discipline unchanged. Ensure the effective rendered prompt states this precedence rather than leaving two competing rules."
    origin: lens:code-scope
    carries: F6
prior:
  - id: F1
    status: resolved
    answer: "RoundFacts names the current focus file and decision record; its tests cover rounds 1 and 2, and the rubric explicitly permits the judge to read both. F12 concerns the separate filename-ownership defect."
  - id: F2
    status: resolved
    answer: "The production campaign Entry seam has a missing-entry test asserting the slug, hardener.packages and resume route; the reader table covers success and failures."
  - id: F3
    status: resolved
    answer: "The preferred Preflight remedy is implemented and tested: an empty-package campaign reads Entry before any agent starts. Seed-time transient failures retain the existing fallback."
  - id: F4
    status: resolved
    answer: "Copies now run through harden-live, and CheckCampaign applies the single-tier check when copies run. Folding the key is expressly allowed by the goal."
  - id: F5
    status: resolved
    answer: "The listed merge-bar, safety-pass, fan exception, suite maintenance, determinism and documentation obligations are present; the enumeration instruction no longer exposes a pre-count. Fixer obligations reach the fix profile."
  - id: F6
    status: open
    answer: "Target instructions now reach the fixer, but omitting the per-finding commit rule leaves the shared grouping exception active. The unchanged generic grouping rule does not amend the decision record's Crucible requirement."
  - id: F7
    status: resolved
    answer: "TestHardenRoundKey drives a Bouncer with a named stub seed stencil and checks that its launched prompt contains the campaign brief."
  - id: F8
    status: resolved
    answer: "hardenPreflight returns the absent gate.yaml error; the reaching test checks its reconcile route."
  - id: F9
    status: resolved
    answer: "Passing Preflight fixtures now seed a sparse gate.yaml, so the harden gate keys come from the template."
  - id: F10
    status: resolved
    answer: "The range contains Loosened-Check trailers identifying the retired checks, the changed requirements and their replacement assertions."
  - id: F11
    status: resolved
    answer: "TestSandboxVerbs now asserts two default bares as well as one single-repository bare and its refs/lym/ records."
---

### [BLOCKING] Charter facts redeclare the owning focus filename

**Location:** internal/hardenshed/campaign_rows.go:24

**Issue:** The new constant repeats `round-%d-focus.md`, already owned by `shedadapters.focusPath` at `internal/shedadapters/round.go:120`.
`RoundFacts` uses its copy while seed, judge, reader and resume paths use the existing builder.
The paths agree today, but a change to the owning builder would leave the charter fact pointing elsewhere.
`pattern/PATTERN-code-conventions.md` requires one declaration referenced everywhere.
The review instructions classify every listed PATTERN violation as BLOCKING; this is a new changed hunk.

### [MEDIUM] The per-finding commit obligation remains undelivered

**Location:** contracts/recipes/hardener-recipe.yaml:133

**Issue:** Prior F6 is partly resolved: the other Hardener obligations now reach the fixer's work statement.
The effective source-scope instructions at `internal/burlerengine/prompt.go:339` still group findings sharing files.
For two findings in one file, they require fixing both before their joint commit.
The retired Crucible template's “Commit per fix” section requires committing each finding before starting the next.
The decision record's Crucible amendment requires carrying that rule over, and its placement amendment preserves the content rules.
No added decision waives this requirement.
The scope and affected-set lenses independently reported this same residual; it is carried once as F6.

## Validation

Reviewed the supplied committed diff against `3a9f8d8419fbcd415992ca15a8c6e9d25822015e..HEAD` and its commit messages.
Read the decision record, board body, round focus, current-round change list, source paths and assertions cited above.
All four lens reports are saved beside this report.

The attempted tier-0 gate over hardenshed, loomcli, shedrecipe, burlerengine and shedadapters started no tests.
The first invocation used the invalid tier spelling `tier0`; the corrected invocation used `--tier 0`.
It failed acquiring a slot because the sandbox makes the hub's `_bolt/.lym/gate` and runproc paths read-only.
No new passing test result or end-to-end campaign result is claimed.
The supplied matching-tree gate reports supplement the source review.
