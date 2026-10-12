---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md, Decisions 2 and 4, Open risks
    summary: Decision 4's bound covers only listed spans and the Edit tool, while Decision 2 moves later cards' whole-file reads ahead of every edit with no bound of its own, and a Write from such a stale whole-file read has no text match to fail on
    fix: In decision-record.md, rewrite Decision 4's Bound so it names everything read before the first edit that an earlier card's edit or a self-fix has since changed, both the listed spans and the whole files Decision 2 reads (uncovered target and Uses files, located unresolved-glyph files). Add to Decision 2 the sentence that the pack-on FRESH-READ fragment tells the fork to change a file it has already edited in this batch only with the Edit tool, never by rewriting it whole with Write from content read before that edit, and add a matching acceptance criterion. Then point Decision 2's own Bound at Decision 4, and widen Open risks bullet 1 so the wide first phase includes the whole files Decision 2 reads, not only resolved spans.
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md, Open risks bullet 2
    summary: The kickstart-pack-measure metric bullet carries no requirement or constraint for the plan writer and restates that board entry's own Next audit paragraph
    fix: Delete the second Open risks bullet (the kickstart-pack-measure calls-before-the-first-edit metric) from decision-record.md. It carries nothing the plan writer acts on, the board entry already holds it, and the support log's Interview item 5 already records the lens finding that raised it.
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md, Auto-mode assumptions bullet 1
    summary: The bullet ends with see the support log, a pointer the plan writer, who never reads support-log.md, cannot follow
    fix: In decision-record.md's first Auto-mode assumptions bullet, delete the trailing clause ending in see the support log, leaving the bullet as the auto-pick of read all first over resolving the pack per card with quarry.
---

### [MEDIUM] The stale-read bound misses the whole-file reads Decision 2 moves up and the Write tool

**Location:** `_lyx/discussion/decision-record.md`, Decisions 2 and 4, Open risks.

**Issue:** The landed pack-on FRESH-READ fragment (`contracts/stencils/webster/webster-fragment-fresh-read-pack.md`) reads a card's uncovered target and `Uses:` files whole "in the turn right after" that card's step 1, so each card's whole-file reads were taken after every earlier card's edits.
Decision 2 moves those reads, and the whole-file reads of located unresolved-glyph files, ahead of the first edit, batch-wide.
Card 3's whole-file read of a file card 1 later edits is then stale in exactly the way a span is.
Decision 2 states no bound for what this lets through.
Decision 4's Bound names only "a later card's span held in context", and Decision 5's guard speaks only of listed lines.
Decision 4 rests the bound on the Edit tool's `old_string` match failing loudly.
A file read whole can also be rewritten whole with the Write tool, which the implementer body (`webster-body-implementer.md`, "Paths you pass to the Read, Write and Edit tools") allows, and Write has no text match.
A Write built from a pre-edit whole-file read silently drops an earlier card's committed change.
Only the batch gate is left to catch it, and the record does not say so.
Open risks bullet 1 counts only resolved-entry spans toward the wide first phase, yet Decision 2 adds whole files to it before any edit.
The rubric's attack-surface rule wants a stated bound for a moved read-before-edit edge and a downgraded guard.

### [LOW] Open risks bullet 2 belongs to the board entry, not the record

**Location:** `_lyx/discussion/decision-record.md`, Open risks bullet 2.

**Issue:** The bullet says the `kickstart-pack-measure` audit's calls-before-the-first-edit metric rises by construction, and that the audit compares other measures.
The `kickstart-pack-measure` board entry's "Next audit" paragraph already says this almost word for word.
Under the completeness-before-leanness test it carries no requirement or constraint the plan writer needs: no card, stencil or test changes because of it.
It is an exclusion, not a relocation, because the support log already records the lens finding that raised it.

### [NIT] Auto-mode assumption 1 points at a file the plan writer never reads

**Location:** `_lyx/discussion/decision-record.md`, Auto-mode assumptions bullet 1.

**Issue:** "see the support log" sends the reader to `support-log.md`, which `Plan-Write` never reads.
The bullet's own text already states the pick, so the pointer adds nothing for the record's one reader.

## Verdicts on the focus directives

- **Decision 4's premise.**
  The record states "batches run one fork at a time and no edit has happened yet" as a premise, and the code supports it.
  `internal/websterengine/doc.go` says forks run "sequentially — one fork at a time, one worktree", and begin-batch builds the pack "immediately before it renders the fork prompt".
  Every listed offset is therefore valid in the first turn.
  The bound for a card editing another card's span, or a self-fix, is stated for spans under the Edit tool.
  The gap is F1's whole files and Write, not the premise.
- **Decisions 4 and 5 together, stale listed lines.**
  For listed spans this is bounded as claimed.
  After an edit, Decision 5 forbids re-reading by a listed line.
  An Edit from a stale span either matches identical text or fails loudly.
  Stale reasoning over unchanged-looking text falls to the card and batch gates.
  The fork made the earlier edits itself, so they sit in its own context.
  Nothing further is unbounded for spans.
- **Decision 2's `plan:` rule.**
  Consistent with the criteria.
  A card that creates a member in a new file has no existing file to read ("every existing file"), and one that adds a member to an existing file reads that file whole as a target.
  A later card's `Uses:` of a `plan:` handle an earlier card of the batch created names code the fork wrote itself, and the kept "read further … whenever the card needs it" sentence covers reading it.
  Its missing bound is F1.
- **Relocation and exclusion.**
  Auto-mode bullet 2 and Open risks bullet 3 point at Decisions 2 and 4 rather than restating them as new constraints; recording an auto-pick and its risk is those sections' purpose, so I raise no finding for either.
  Open risks bullet 2 is excluded as F2.
  No Decision carries only deliberation narrative: Decision 2's "This extends the board entry" sentence is the scope label the earlier goal-scope lens asked for.
- **Acceptance criteria against Decisions.**
  Decision 1 is covered by criteria 1 and 2, Decision 2 by criteria 3 to 5, Decision 3 by criterion 6, Decision 4 by criteria 7 and 11, and Decision 5 by criterion 8.
  Criteria 9 and 10 assert what the first Constraint and Scope's Out list decide.
  Criterion 5's quarry-unavailable case is a pack-on case, because an index error keeps a degraded pack (`doc.go`, "An index error keeps the degraded pack the index returned").
  No criterion asserts something nothing decides.
