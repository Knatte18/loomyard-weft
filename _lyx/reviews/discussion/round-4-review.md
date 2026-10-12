---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:73
    summary: loom.discussion_producer stays open to the local layer on the false premise that it picks only the writer, yet setting it from twill to single switches off the twill writer's lens-fanned review of the decision record
    fix: In D4, add discussion_producer to loom's LocalDenied list (line 69), replace line 73 with the reason that twill makes the Discussion-Write writer fork one reviewer per lens of burler's twill-discussion fan over its own record, so a local value of single or seats switches off a hub-chosen review untracked, and add loom.discussion_producer to the LocalDenied examples of the acceptance row at line 223; update Q5 in support-log.md only if its wording no longer covers it.
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:28
    summary: D1 does not say which .env expands a ${env:...} marker once layers stack, and the repo default sits under the board dir while the local file sits under the anchor, so D6's claim that the strip never changes a held key's resolution depends on an unmade choice
    fix: Add to D1 a sentence stating that the layers are merged unexpanded and the merged document is expanded once, through envsource.Build of the reading worktree's anchor for a per-worktree module and of the board dir for a hub-wide module, and that no layer is expanded against its own directory's .env; add an acceptance row that a ${env:...} marker the worktree file inherits from the repo default expands against the reading worktree's .env.
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:231
    summary: The strip's acceptance row covers only a worktree literal over a marker below, not the reverse D6 line 102 also rules on, a worktree marker over a literal below that equals its expansion
    fix: Extend the acceptance row at line 231 with the reverse case, a ${env:...} marker in the worktree file over a literal in the repo default equal to the walk's expansion of that marker, which is kept.
---

### [BLOCKING] `loom.discussion_producer` opens a review switch-off to the local layer

**Location:** `_lyx/discussion/decision-record.md:73`

**Issue:** D4 keeps `discussion_producer` open "because it picks the writer, not a reviewer".
The value `twill` does more than pick the writer.
`internal/loomengine/twill.go` renders the twill directive from burler's `twill-discussion` fan, and its `twillFan` comment says the twill writer's forks "review the decision record through" that fan's lenses (`premises`, `goal-scope`, `board-fit`, `pattern-fit`, `scenarios`, `leanness` in `internal/burlerengine/template.yaml`).
The template comment on `discussion_producer` in `internal/loomengine/template.yaml` says the same: twill "forks itself once per lens of the twill-discussion fan in burler.yaml to review its record before it finishes".
A hub that sets `twill` in the repo default therefore gets a lens-fanned review, and a pair's local file setting `single` turns it off, untracked and unseen at landing.
That is the case D4 already denies for `discussion_fan` and `plan_fan`, which turn "a hub-chosen fanned review back to a solo one", and which D4's category test names ("switches off or hollows a guard, including a review ... and the prose or lens set a reviewer is given").
The premise is false, the category test applied as written denies the key, and D4's Bound (line 79) and R3's bound to "session tuning" (line 209) are untrue while it stays open.
Round 3's review classed `discussion_producer` as acting on the writer and its fix left the key open "if that is the intent"; neither looked at the twill writer's forks, and no fixer report disputed the point, so this is a new finding, not a re-raised dispute.
Every other per-worktree key I tested against D4's categories lands where the lists put it:
loom's `review_max_bounces` is refused below 1 by `loomrecipe` (`checkReviewBudget`), webster's `verify_gate_attempts` and darn's `verify_attempts` are refused at or below 0 at load or build, and reed's `operator_quiet_s` takes the default at 0, so none of them can switch a gate off.

### [MEDIUM] Marker expansion across stacked layers is undecided

**Location:** `_lyx/discussion/decision-record.md:28`

**Issue:** Today `configengine.load` resolves one file and expands its markers once with `envsource.Build(baseDir)`, which reads `<baseDir>/.env` under the process environment (`internal/configengine/config.go`, `internal/envsource/envsource.go`).
D1 and D3 stack files from two roots, the repo default under `<BoardDir>/_lyx/config/` and the worktree and local files under the anchor, and no Decision says whether a marker is expanded per layer against that layer's directory or once after the merge against the reading worktree's.
D6 rests on the second reading: it compares unexpanded nodes (line 102) and states the strip never changes a held key's resolution, which holds only if a marker stripped into the repo default still expands against the worktree's `.env`.
A plan writer who picks per-layer expansion against the board dir's `.env` breaks that claim for every worktree whose `.env` binds a marker, such as `LYX_SHUTTLE_CLAUDE`, and nothing in the record forbids that pick.
The same gap leaves open which `.env` expands a marker written into a hub-wide module's local layer.

### [LOW] Strip acceptance row covers one direction of the marker comparison

**Location:** `_lyx/discussion/decision-record.md:231`

**Issue:** D6 line 102 says a marker equals only the same marker text, which rules both ways.
The acceptance row pins only the literal-over-marker case.
The reverse is the common one after migration: a worktree file materialized from the template holds `claude: ${env:LYX_SHUTTLE_CLAUDE:-}`, while the prime's file, seeded as the repo default, may hold a literal path equal to the walk's expansion.
An expanded comparison would strip the worktree's marker and silently drop its `.env` binding, and no row catches that.

## Focus verdicts

- D4 category test against the lists: I tested every key of every module's template.
  The lists match the test except `loom.discussion_producer` (F1).
  `loom.friction` (empty turns Tier 2 self-reporting off) and `loom.discussion_interactive` meet no category: neither gates a landing nor switches off a guard or a review.
- D4 Rationale and R3 on `.env`: the record now states the `.env` route (line 83, R3, and the parent's added entry), and R3's precedence claim, process environment over `.env`, matches `envsource.Build`.
  R3's bound to session tuning is false only through F1.
- D6 unexpanded comparison: stated at line 102 in both directions; the acceptance row covers one (F3).
- D6 seeded-default keys: the narrowed claim holds.
  `configsync` materializes an absent per-worktree file from the template, so a worktree file lacks only keys beyond the template, which D6 lines 105–107 report as `inherited` and the acceptance row at line 232 pins; it depends on F2's expansion choice for marker-carrying keys.
