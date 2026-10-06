---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: _lyx/plan/14-reed-shuttle-prune.md, _lyx/plan/16-fabric-geometry-prune.md
    summary: Cards 14 and 16 each pair one slow fixture-merge package with unrelated near-instant unit packages, so one reviewer cannot judge the card against one concern
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/01-measure-before.md, _lyx/plan/00-overview.md prune-procedure step 2
    summary: Card 1 runs the failed entries alone and keeps the verdicts for the package cards, but step 2 makes every package card redo that run and nothing names where card 1's verdicts go
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/plan/13-review-gates-prune.md, _lyx/plan/00-overview.md Card Index
    summary: Card 13's title, Card Index line and opening Intent sentence do not name internal/statuscommit, which the card targets
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/plan/21-rerun-measure-after.md
    summary: The card cites Decision 1's parent addition for cmd/testtiming being core's, but that is a separate parent addition, not part of Decision 1
---

### [LOW] Cards 14 and 16 mix a slow fixture-merge package with unrelated unit packages

**Location:** `_lyx/plan/14-reed-shuttle-prune.md`, `_lyx/plan/16-fabric-geometry-prune.md`

**Issue:**
Card 14 targets `internal/reedcli` (228.16s in `docs/benchmarks/test-redundancy.md`, almost all of it tmux, the plan's tmux-tier scenario-merge target), `internal/reedengine/render` (0.01s), `internal/shuttlecli` and `internal/shuttleengine/claudeengine` (0.41s).
The reed half needs the tmux-tier scenario rules.
The shuttle half needs the provider-seam rule and the whole skill-loading carve-out against `skills-in-one-turn`.
Those are two different concerns, and the carve-out is easy to miss while the reviewer's attention is on the 228s package.
Card 16 targets `internal/fabriccli` (17.82s, the main `hubforge` scenario-merge lead) and `internal/pairteardown` (5.13s) together with `internal/hubgeom`, `internal/standalonegeom`, `internal/standalonestate`, `internal/lyxcwd` and `internal/weftname`, which are pure geometry packages (0.01s to 1.83s).
The two halves share no fixture, rule or reviewer question.
The other multi-package cards bundle packages of one family or all small ones (cards 3, 9, 15, 17 through 20), so this is a defect only in these two.
The leaf-package rule is carried correctly where it applies (`standalonestate` in card 16, `gitkit` in card 17, the five others in card 20), and splitting must keep it on the card that holds `standalonestate`.

**Fix:**
Split card 14 into a reed card (`internal/reedcli`, `internal/reedengine/render`) and a shuttle card (`internal/shuttlecli`, `internal/shuttleengine/claudeengine`, with the skill-loading carve-out), and card 16 into a fabric-hub card (`internal/fabriccli`, `internal/hubforge`, `internal/pairteardown`) and a geometry card (`internal/hubgeom`, `internal/standalonegeom`, `internal/standalonestate`, `internal/lyxcwd`, `internal/weftname`).
Renumber the later cards and update every file name, `Card Index` entry, `Commit:` subject, `Applies to:` range and card-number citation to match.

### [LOW] Failed-entry verdicts are produced twice, with no hand-off from card 1

**Location:** `_lyx/plan/01-measure-before.md`, `_lyx/plan/00-overview.md` (`prune-procedure` step 2)

**Issue:**
Card 1 runs each in-scope `failed` test alone and says "keep the verdicts for the package cards", but names no place to keep them.
`.scratch/` is the only candidate and may be wiped.
Step 2 of `prune-procedure` has every package card rerun its own `failed` entries under Decision 8 before touching them, and the package cards (2, 7, 9, 13, 14, 15, 19) name them again.
Card 1's run therefore has no consumer.

**Fix:**
Drop the failed-entry run from card 1, so Decision 8 is performed once, in the package cards, per step 2.
Card 21's check of the final rerun against the commit messages' tool-artifact notes is unchanged.

### [NIT] Card 13's label omits `internal/statuscommit`

**Location:** `_lyx/plan/13-review-gates-prune.md`, `_lyx/plan/00-overview.md` Card Index

**Issue:**
The card's `Edit:` list includes `internal/statuscommit#`, but the Card Index line ("burler, treadle, review, preflight and verify packages") and the Intent's first sentence ("review, gate and verify packages") name no status-commit package.

**Fix:**
Name `statuscommit` in the Card Index line and in the opening Intent sentence.

### [NIT] Card 21 cites the wrong decision for `cmd/testtiming`

**Location:** `_lyx/plan/21-rerun-measure-after.md`

**Issue:**
The card says `cmd/testtiming`'s tests are core's under "Decision 1's parent addition".
The decision record adds that rule as its own "cmd/testtiming's tests are core's" addition after Discussion; Decision 1 covers the six core directories and `internal/testkit/`.
`ownership-boundary`'s rationale cites it correctly as a parent addition.

**Fix:**
Cite it as "the parent's `cmd/testtiming` addition".

## Checked, no finding

- Card numbers cited by number name the card that does the stated work: `rerun-measure-after` rewrites the report and is the card the other cards call "card 21", card 1 is the before measurement, and the `Applies to:` ranges (`2–21`, `2–20` plus 21) match the cards that edit test files.
  Card Index entries, file names and `Commit:` subjects agree for all cards.
- The keep-reason closed set is one list in `prune-procedure` step 6: the four classes of Decision 10 plus `skills-in-one-turn`.
  Step 8, `ownership-boundary`, card 2, card 14 and card 21's check use it consistently, and an arbitrary annotation satisfies none of them.
- Every package of the report except the six core packages and `internal/testkit/...` appears in exactly one card's `Edit:` list, and all files and failed-entry tests the cards name exist.
- Card 21 leaves the `cmd/testtiming` report section unspliced and its keep check waived.
  The decision record bars edits under `cmd/testtiming`, tests included, so those candidates cannot be annotated, and core changes that package's redundancy mode; the exception is a sound reading, not a defect.
