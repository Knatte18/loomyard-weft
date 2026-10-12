---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md, Decisions, D4 (and D3's one-turn claim)
    summary: D4's pack-on FRESH-READ tells the fork to read, in the same turn as the card file, any file the card targets or uses that no span covers, but those files are named only in the card file, which that turn is reading, so the read cannot happen in that turn
    fix: In D4, replace "and reads in the same turn any file the card targets or uses that no span covers" with "and, in the turn right after, before any edit, reads whole every file the card targets or uses that no listed span covers (a plain path, a self glyph's file, an unresolved member glyph once located)". In D3's rationale, change "makes the card file and its spans one turn instead of two" to say the card file and its listed spans are one turn, and files only the card names follow in the next turn. Add an Acceptance criterion that the pack-on FRESH-READ text has the fork read, before any edit, the target and Uses files no listed span covers.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md, Decisions, D3 and D4; Open risks
    summary: D3's step-1 read takes every span by offset at the card's start, with no carve-out for a file an earlier card of the batch edited, yet D4 calls that read the card's fresh read, so for a later card in an edited file the fresh read is of shifted lines and the find-by-signature rule only arrives at the later locate step
    fix: In D3, add to the pack-on card-read fragment that a span in a file an earlier card of this batch edited is not read by offset; that file is read whole in the same turn and the member found in it by its listed signature. In D4, say the fresh read of such a file is that whole-file read. Then replace the Open risks entry with the residual risk only (a var or const signature is one line, so two members with the same first line in one file can still be confused), or delete it if none remains.
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md, Decisions, D3
    summary: D3 replaces step 1 with a pack-on fragment but does not say whether that fragment keeps step 1's other two sentences, the card file being the whole instruction and the Intent fallback to the overview's Card Index
    fix: In D3, after "The pack-on fragment tells the fork ...", add that the pack-on fragment keeps step 1's two following sentences verbatim (the card file is the card's whole instruction; an empty Intent field falls back to the Card Index line in `_lyx/plan/00-overview.md`), so only the read sentence changes.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, Decisions, D1 and the quarry-unavailable Acceptance criterion
    summary: D1 says that on ErrQuarryUnavailable each card lists its selected refs as unresolved with the error, as today, but today a plan handle keeps its own handle reason under that error and only member glyphs carry the error
    fix: In D1 and in the matching Acceptance criterion, change "each card lists its selected refs as unresolved with the error" to "each card lists its selected member glyphs as unresolved with the error and its `plan:`-handle refs with the handle reason".
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md, Decisions, D2
    summary: D2 covers a card with no member glyph (empty-pack line) but not a card that names member glyphs none of which resolved (quarry unavailable or every glyph not found), where today's section renders the entries as none; whether the card's heading shows none or no entries line is left open
    fix: In D2, add one sentence that a card naming member glyphs none of which resolved renders `none` in place of its entries, followed by its unresolved list, and extend the per-card Acceptance criterion with that case.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md, Interview, step 3
    summary: Interview step 3 says a grouped spec carries the spec's own text, while D5's rationale and quarry v0.2.1 (goGroupedConstOrVarSymbols prepends the var or const keyword to the spec's text) say the first line reads var name or const name in both forms
    fix: In support-log.md Interview step 3, replace "grouped specs carry the spec's own text, so a first line names the member" with "a grouped spec's signature is its keyword prepended to the spec's own text, so in both forms the first line reads `var name …` or `const name …`".
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md, Scope, Out, the kickstart-pack-measure bullet
    summary: The two sentences under the kickstart-pack-measure bullet, old pack-on counts not being a baseline and queuing being the operator's call, carry no requirement the plan writer acts on; they are deliberation about a different task
    fix: In decision-record.md Scope Out, cut the bullet down to "Running the with/without measurement (`kickstart-pack-measure`)." and move the two removed sentences into support-log.md's Interview step 5, under the board-fit MEDIUM entry that already records them.
---

### [BLOCKING] D4's same-turn read of uncovered files is impossible

**Location:** `_lyx/discussion/decision-record.md`, Decisions, D4 (and D3's rationale).

**Issue:** The pack carries only member glyphs and `plan:` handles; `KickstartPack` (`internal/planglyph/pack.go`) leaves plain paths, self glyphs and Create targets out.
A card's file-level targets and `Uses:` paths are therefore named only in the card file.
D3 makes the card file and its spans one parallel turn at the card's start, and D4 says that same turn also reads "any file the card targets or uses that no span covers".
The fork cannot know those files until the card file it is reading in that turn comes back.
Written as decided, the pack-on FRESH-READ stencil asks for an impossible read.
It also leaves unspecified when a fork re-reads a plain-path target fresh, which is the very file kind the old whole-file rule protected.
On the focus question about D4's bound: it covers member-glyph targets, but plain-path and self-glyph files fall through this gap.

### [MEDIUM] The step-1 offset read has no carve-out for files the batch already edited

**Location:** `_lyx/discussion/decision-record.md`, Decisions, D3 and D4; Open risks.

**Issue:** D3's step 1 reads every listed span by offset and limit before anything else.
The rule that spans in an already-edited file are found by signature sits in the locate fragment, which the body renders at step 2, after that read.
D4 then calls the step-1 turn the card's fresh read, so for a later card whose spans sit in a file an earlier card edited, the declared fresh read lands on shifted lines.
The Open risks entry names the shift but no Decision bounds it.
The fork itself made the earlier edits, so it knows which files are affected, and a decision can route those files to a whole-file read in the same turn.

### [LOW] D3 leaves step 1's remaining sentences undecided

**Location:** `_lyx/discussion/decision-record.md`, Decisions, D3.

**Issue:** In `contracts/stencils/webster/webster-body-implementer.md`, step 1 has three sentences: the read, "It is your whole instruction for that card.", and the Intent fallback to the Card Index.
D3 says the pack-off fragment is today's text verbatim but describes only the read for the pack-on fragment.
That leaves the plan writer to decide whether the Intent fallback survives under pack-on.

### [LOW] D1's quarry-unavailable listing misstates today's handle reason

**Location:** `_lyx/discussion/decision-record.md`, Decisions, D1, and the quarry-unavailable Acceptance criterion.

**Issue:** In `KickstartPack`, the `ref.isHandle` case runs before the `resolveErr != nil` case, so a `plan:` handle keeps `packHandleReason` even when quarry is unavailable.
D1 says each card lists its selected refs with the error, "as today", and the criterion repeats it.
A plan writer following the text literally would change the handle reason.

### [LOW] D2 leaves open what a card with only unresolved member glyphs renders

**Location:** `_lyx/discussion/decision-record.md`, Decisions, D2.

**Issue:** `renderPackSection` today renders `none` for the entries when member glyphs are named but none resolved, and the empty-pack line only when no member glyph is named.
D2 decides the empty-pack case per card but says nothing about the "named, none resolved" case, which every card hits when quarry is unavailable.
On the focus question about cards with no resolved spans: D3's "read every span the pack lists" reduces to the card file alone in that case without contradiction, but D2 has to pick what the heading shows.

### [LOW] The support log's grouped-spec premise disagrees with D5

**Location:** `_lyx/discussion/support-log.md`, Interview, step 3.

**Issue:** In quarry v0.2.1, `goGroupedConstOrVarSymbols` (`internal/engine/golang.go`) builds the signature as `keyword + SignatureCut(spec, nil, src)`, and the ungrouped path cuts from the declaration's own `var`/`const` keyword.
D5's rationale states this correctly and is what the `var`/`const` Acceptance criterion tests, across the ungrouped and grouped forms.
Interview step 3 still says "grouped specs carry the spec's own text", the pre-fix premise the support log's own lens entry ("premises LOW, grouped signature carries the keyword") says was corrected.

### [LOW] Scope Out carries measurement deliberation the plan writer never acts on

**Location:** `_lyx/discussion/decision-record.md`, Scope, Out.

**Issue:** Under the completeness-before-leanness test, neither sentence states a requirement or constraint for this task's plan.
Whether old pack-on counts form a baseline, and who queues the measurement, belong to `kickstart-pack-measure`.
D4's rationale already carries the requirement that does matter here, keeping the pack-off prompt byte-identical for comparability.

## Rejected

- D3's bench-card rationale: it is Rationale, which `## Decisions` admits by contract, and the requirement it motivates is already stated in D3's Decision text; not a defect.
- Auto-mode assumptions section: the section is required by the writer's contract, and its one line carries no deliberation; not a defect.
- D2's heading rule: `RenderForkPrompt` already holds `planDir` and `promptWorktreeRoot`, and `renderCardPointers` spells the path as `masterPlanDirDisplay(promptWorktreeRoot, planDir)` plus `filepath.Base(c.SourcePath)`, so "same path, same display rule" fixes one spelling under both geometries; not a defect.
- D1, D2 and D6 together: `buildKickstartPack`'s log counts come from the pack today, and D6 states that a ref unresolved under two cards counts once, as a distinct batch ref. The per-card fan-out does not conflict with that; not a defect.
- Acceptance criteria against Decisions: D1 to D6 each have a behavior criterion, and no criterion asserts something no Decision or Scope line states. The one uncovered clause, D4's uncovered-file read, is handled under F1.
