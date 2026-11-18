---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:53
    summary: D2 claims the viewer ends on the beam overview window after the move, but no step of the mechanism selects it and reed's rules move the viewer elsewhere
    fix: In D2, replace the sentence "The viewer ends on the beam overview window, as after today's replace." with a decided mechanism and its bound, for example that ensureRunsStrand, after a successful runs-strand add that followed a move which ended the viewed window, selects the runs strand's overview window through a reed op that selects a strand's window (name the op as new if reed has none), and that it selects nothing when the viewer was on another window; then reword the criterion at line 104 to that behaviour and say in Constraints which test tier pins it, since viewer selection is tmux-tier reed behaviour. If the writer drops the claim instead, delete the sentence and the criterion at line 104.
  - id: F2
    severity: LOW
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md:93
    summary: Parent commit 7c23591, landed on main and on the parent beam branch after this worktree's base, already fixes this bug with a contradicting design; only the parent can say whether this bench arm redesigns from its base without reconciling
    fix: Goes to the parent, not the fixer. The parent answers whether this arm, by design of the plan-writer-explorer-forks bench (frozen inputs from the reference run's base commit), redesigns the bug from base f98a1ba0 without reconciling with 7c23591; the answer is recorded with lym loom decision add, and the Open risks bullet at lines 93-95 is then dropped.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:63
    summary: D3 says Title names an own-window strand placed apart, and also that a strand in a shared window group ignores Title, so a grouped strand that reed places apart in its own window falls under both rules
    fix: In D3, state which rule a strand declaring a window group but placed in a window of its own (group full or over window_share_max) follows, for example "A strand that shares its group's window ignores Title; one of a group placed in a window of its own follows the own-window rule", and make the D3 criterion at line 107 name that case.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:109
    summary: The D4 criterion draws the own-run line while the run is live, but D4 draws it while the run has a seed and a status file that is not done, which also covers a halted or paused run
    fix: In the criterion at line 109, replace "while the worktree's run is live" with "while the worktree's run has a seed and a status file that is not done", matching D4 at lines 72-74.
  - id: F5
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:63
    summary: D3 lets any own-window strand name its window by Title but does not state the facts that make a duplicate or colliding name harmless, so Plan must re-derive them
    fix: Add one sentence to D3 stating that tmux allows two windows of one name, that reed addresses an own window by its window id or a per-call temporary name and finds the shell window by its console pane (internal/reedengine/doc.go window-order and detached-window paragraphs, internal/reedengine/windoworder.go), and that OpenWindow's same-name lookup skips windows carrying @lym_own, so a Title shared by two strands only gives two buttons of one name.
  - id: F6
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:83
    summary: The Constraints bullet and D3's rationale cite PATTERN-no-hard-coded-choice for a recipe name, but that entry governs language, engine and branch choices only
    fix: In the Constraints bullet at line 83, drop the PATTERN-no-hard-coded-choice label and state the rule as the record's own (the window name is read from the run's seed and placement is keyed on the runs strand's presence, never on a recipe name); in D3's rationale at line 68, drop the clause "per PATTERN-no-hard-coded-choice".
---

### [BLOCKING] D2's viewer end state has no mechanism and contradicts reed's focus rules

**Location:** `_lym/discussion/decision-record.md:53`, with the criterion at line 104.

**Issue:** D2 case 1 replaces the overview `loom-status` strand with an own-window `KeepViewer` spec, and only then adds the runs strand.
Each step leaves the viewer somewhere other than the beam overview:

- `ReplaceStrand` with a non-overview spec calls `killStrandPanes(..., selectOnEnd=true)` (`internal/reedengine/replace.go:70-71`).
  A viewer on the ending window moves to `relevantWindow`: the newest live agent window, else the overview, else none (`internal/reedengine/ownwindow.go:456-470`).
  No overview window exists at that moment.
- The new checklist window sets `KeepViewer`, so it is never selected (`internal/reedengine/ownwindow.go:479-480`).
- `AddStrand` of the runs overview never selects its window ("An overview window is never selected here", `internal/reedengine/spawn.go` doc of `openOwnWindowLocked`).
- Today the viewer reaches the new overview window only because the replacement is itself an overview strand (`endedCurrent && replacingWithOverview`, `internal/reedengine/replace.go:97-98`).
  D2's move no longer takes that path, so "as after today's replace" is a false premise.

The criterion "After the move the viewer is on the beam overview window" cannot pass without a step no Decision designs.
It also sits outside the record's own `PATTERN-test-speed` constraint, which puts every criterion but reed's window naming in pure or fake-seam tests.
The bound-breaker, existing-paths and interactions lenses each filed this at BLOCKING and walkthrough at MEDIUM.
It is kept once at BLOCKING because Plan would otherwise have to design a reed selection step on its own.

### [LOW] Open risk on parent commit `7c23591` is a question for the parent

**Location:** `_lym/discussion/decision-record.md:93`.

**Issue:** The record's facts check out against the repository:

- `7c23591` ("Show a beam's pawl checklist in a second overview window") is on `main` and on the parent beam's branch `plan-writer-explorer-forks`.
- It is not an ancestor of this worktree's HEAD `f98a1ba0`, which is itself an ancestor of `main` and predates `7c23591` by about two hours.
- Its message states a design that contradicts D1 to D3: overview strands ordered through a new `Display.Order`, `statusStrandAddSpec` always returning an overview display titled by the recipe, and no `loom-status` lookup in `ensureRunsStrand`.

This is a landed parent commit that contradicts this record's decisions, which the rubric classes as `decision`.
The parent beam's entry suggests the answer: a bench run from "frozen inputs", starting from "the base commit" of a reference run.
But neither the record's wording nor a `Decision for wxb-plan-with:` board note settles it, and `lym board list` shows no such note.
So the question stays with the parent rather than being dropped by the fixer.

### [LOW] D3 is ambiguous for a grouped strand placed apart

**Location:** `_lym/discussion/decision-record.md:63`.

**Issue:** Reed's "placed apart" path is a grouped strand opened in a window of its own (`internal/reedengine/spawn.go:185-186`), named today by `strandLabel`.
D3 puts "an own-window strand placed apart" under the Title rule and "a strand in a shared window group" outside it.
Those two sentences disagree for that path, and the D3 criterion inherits the ambiguity.
The walkthrough lens found the same gap; the two reports are merged here.

### [LOW] D4 criterion's condition differs from D4's

**Location:** `_lym/discussion/decision-record.md:109`.

**Issue:** D4 draws the line while the run has a seed and a status file that is not done.
The criterion says "while the worktree's run is live".
A halted or paused pawl run satisfies D4's condition but arguably not the criterion's.

### [LOW] D3 leaves its window-name collision facts unstated

**Location:** `_lym/discussion/decision-record.md:63`.

**Issue:** D3 extends `Title` to every own-window strand, not only `loom-status`.
It holds because:

- tmux allows duplicate window names;
- reed targets an own window by id, or by a per-call temporary name (`internal/reedengine/doc.go`, detached-window paragraph; `internal/reedengine/window.go`);
- reed finds the shell window by its console pane (`internal/reedengine/windoworder.go`);
- `OpenWindow`'s same-name lookup skips `@lym_own` windows.

The record states none of this, which the focus asked to check.

### [NIT] `PATTERN-no-hard-coded-choice` cited outside its scope

**Location:** `_lym/discussion/decision-record.md:83`, and D3's rationale at line 68.

**Issue:** `PATTERN-no-hard-coded-choice` and its background file cover a language, an agent engine and a branch.
A recipe name is none of these.
The rule the record states is sound, but labelling it with that entry tells Plan that the entry binds here.

## Rejected

- Walkthrough's MEDIUM rating of the viewer finding: merged into F1 at BLOCKING, since the claimed end state needs a design step no Decision gives.

## Holistic notes

- **Relocation test:** the D1 to D4 rationales and the Open risks bullet carry design reasons or the parent question, not interview narrative, so nothing needs moving to `support-log.md`.
- **Attack surface:**
  - D2 states the bypasses it adds: the replace of `loom-status`, the fall-back to today's replace, and their warnings and ways forward.
  - D3 states its fall-back to the role name for an empty or reserved `Title`.
  - Nothing in either is left unbounded beyond F3.
- **Strand states:** D2 cases 1 to 3 and the failure bullets cover no, live, dead and own-window `loom-status`.
  Every failure end state has a criterion.
- **Acceptance criteria:**
  - D1 to D4 are each named.
  - The doc criterion is marked `Review:`.
  - The D4 criterion is checkable over `overviewSource.frame` with injected pollers, without a live reed.
