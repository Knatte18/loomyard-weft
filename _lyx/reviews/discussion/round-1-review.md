---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D8 — Re-measurement hand-off)
    summary: D8 rests on a false premise; the task summary that becomes the landing commit message is webster's summary.md, which the Master writes strictly from batch digests, so no plan or card can make it name the re-measurement
    fix: In decision-record.md, rewrite D8 so its carrier is one that exists. Drop the claim that the task's summary names the check. State instead that the re-measurement is carried past the board entry's close by a board note filed through lyx board by the orch at landing (the carrier support-log.md Q9 already names), and that no card implements D8. Keep D8's content (tools/forkaudit over post-deploy Webster runs, per-fork total reads and failed Edit calls against the 2026-10-12, -13 and -15 audits, not calls before the first edit, the Master's own reads as the D1 compliance check), and add the board entry's expectation that forks explore more by construction, so a rise in per-fork reads is expected and not a regression; that expectation currently lives only in support-log.md's question ledger. Update the D8 rationale and support-log.md's Q9 ledger line to match.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md (D1, D4, Auto-mode assumptions)
    summary: The pattern/ read set is worded three ways, and D4's claim that the recovery strand reads the same set as the Master is false, since the recovery prompt renders the implementer directive and the Master the orchestrator directive, each with its own filter
    fix: In decision-record.md D4, replace the pattern/ wording with the set the recovery prompt's own constraints section points at (the implementer directive, filtered by what the strand is about to change), and delete the clause the same set the Master reads. In D1, keep the constraints-section wording as the one definition of the Master's set and add that, with card files banned, the Master applies the orchestrator directive's filter (what the forks will do) from 00-overview.md's Card Index and Shared Decisions alone. Delete the first Auto-mode assumptions bullet, which then restates D1.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md (Acceptance criteria, third bullet; D1)
    summary: The empty-pattern_directive criterion cannot be met as worded by a conditional-free template; the rendered prompt still names the pattern/ background files, conditioned on the constraints section, so it does not tell the Master to read nothing but the plan and a findings file
    fix: In decision-record.md, replace the third acceptance criterion with one a conditional-free template can meet and a test can check, e.g. The master prompt rendered with an empty pattern_directive contains no constraints section, and every mention of the pattern/ background files in it is conditioned on that section being present; it names no read beyond the plan directory and a named gate findings file. Leave D1's vacuous-wording sentence as the decision this criterion checks.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md (D3 — FRESH-READ fragments)
    summary: D3 leaves the replaced span and the kept text ambiguous; the kept sentence Only your own reads, taken now, are current; content you merely inherited is not duplicates the new wording and contradicts D1's premise that the inherited plan and rules stay current, and the plain fragment's re-read presumes a prior read the cold recovery strand and a post-D1 fork never made
    fix: In decision-record.md D3, name the replaced span exactly in each of the three live fragments - the sentence Inherited context can be stale, the sentence A file you looked at during orientation ..., and the sentence pair Only your own reads, taken now, are current; content you merely inherited is not - and state that one replacement statement covers all of them - no code reaches you from Master, the plan and rules you hold stay current, and only your own reads of code taken in this batch are current. Add that in webster-fragment-fresh-read-plain.md the verb re-read becomes read, so the text holds for the cold recovery strand. Keep the rest of each fragment unchanged as D3 says.
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (Open risks, second bullet; D1)
    summary: D1's read ban is prompt-only and bypassable, and neither D1 nor its Open risks bullet states what a non-compliant Master read lets through or what bounds it; the bound appears only in the in-flight-deploy bullet
    fix: In decision-record.md, extend the second Open risks bullet with the consequence and bound - a Master that reads code anyway hands those reads to later forks as stale context; the bound is D3's fresh-read rule, under which each fork reads every file it edits itself before the first edit, so a stray Master read costs context, not correctness; D8's check detects it.
---

### [BLOCKING] D8's carrier, the task summary, cannot carry the re-measurement

**Location:** `_lyx/discussion/decision-record.md`, D8.

**Issue:** D8 says "the task's summary, which becomes the landing commit message, names the post-landing check".
That summary is webster's `summary.md` (`internal/summaryparser` declares it; `internal/landingshed` turns it into the landing message).
The Master writes it, and `webster-template-master.md:243-245` restricts it to "a prose narrative built strictly from the minimal reports and digests you actually read".
No card can write it: forks are told never to write the Master's contract files, and Go only appends audit and triage sections.
No plan text reaches it either, since the Master may not put plan content into the summary.
So the plan writer has no lever to implement D8, and the board entry's "re-measure afterwards" is left with no carrier.
The board entry's expectation that forks explore more by construction, which governs how the re-measure is read, sits only in `support-log.md`'s ledger, which `Plan-Write` never reads.

### [MEDIUM] Three wordings of the pattern/ read set, and D4's "same set" is false

**Location:** `_lyx/discussion/decision-record.md`, D1, D4 and the first Auto-mode assumptions bullet.

**Issue:** D1 names "the `pattern/` background files the constraints section above it points at".
D4 names "the `pattern/` background files the plan's decisions and its cards touch, the same set the Master reads".
The Auto-mode bullet names "the `pattern/` background files the orchestrator pattern directive points at".
D1 and the Auto-mode bullet agree, since the Master's constraints section is the orchestrator directive (`render.go:626`).
D4 does not: `RenderRecoveryPrompt` renders `pattern.RoleImplementer` (`render.go:493`), whose filter is "touches what you are about to change" (`pattern-directive-implementer.md:17`), while the orchestrator's is "touches what the forks you are about to spawn will do" (`pattern-directive-orchestrator.md:17`).
The Master's set covers the whole plan and the recovery strand's covers one batch, so they are not the same set.
D4's own wording is also a third definition beside the directive the recovery prompt already carries, and the plan writer cannot tell which one the reworded orientation step should name.
D1 also leaves unstated how the Master applies the orchestrator filter once card files are banned: only the overview tells it what the forks will do.

### [MEDIUM] The empty-pattern_directive criterion contradicts the conditional-free template

**Location:** `_lyx/discussion/decision-record.md`, third acceptance criterion, against D1.

**Issue:** D1 says the template has no conditionals, so the read-set wording names the rules through the constraints section and reads as vacuous when the pattern directive renders empty.
The rendered prompt with an empty `pattern_directive` therefore still names the `pattern/` background files, conditioned on a section that is absent.
The criterion says that prompt "tells the Master to read nothing but the plan and a named findings file".
Read literally, a conditional-free template cannot meet it; read as "vacuously", it adds nothing a test can tell apart from the second criterion.
A plan writer turning it into a `template_test.go` row has no checkable assertion to write.

### [MEDIUM] D3's replaced span is ambiguous and the kept text contradicts D1

**Location:** `_lyx/discussion/decision-record.md`, D3.

**Issue:** D3 quotes the replaced text with an ellipsis and says "the rest of each fragment ... is unchanged".
Each of the three live fragments goes on with "Only your own reads, taken now, are current; content you merely inherited is not." (`webster-fragment-fresh-read-pack.md:8-9`, `-pack-card.md:8-9`, `-plain.md:9-10`, pinned at `claims_test.go:657`).
If that sentence is kept, D3's new wording ("only the fork's own reads, taken in this batch, are current") states it twice.
Kept, it also says inherited content is not current, which after D1 means the plan and the rules, the very reads D1's rationale calls fixed during a run.
The plain fragment, which the cold recovery strand also renders, says "re-read — in THIS turn — every card file": the strand inherited nothing and, after D4, its orientation reads no code, so there is nothing to re-read.
D3 requires the wording to hold for the cold strand but leaves this verb in the "unchanged" rest.
I found no Decision resting on an unstated premise about which fragment serves which layout: `renderReadRules` (`render.go:192-215`) maps pack-off to plain, file to pack and card to pack-card, matching the criterion's three live layouts, and the recovery prompt takes plain (`render.go:488`), as D3 states.

### [LOW] D1's prompt-only ban states no bound on what it lets through

**Location:** `_lyx/discussion/decision-record.md`, the second Open risks bullet and D1.

**Issue:** The read ban is enforced by the prompt alone (Scope Out: no hook or audit check).
The second Open risks bullet says only that D8's check shows whether the Master complies.
Neither it nor D1 says what a non-compliant read lets through (stale code reaching later forks) or what bounds it.
The bound exists, in D3's fresh-read rule, but the record states it only for the in-flight-deploy case in the third bullet.
The plan writer reads the whole record, so this is a gap in statement, not a missing decision.

## Focus departures

- "Completeness-before-leanness on Scope's Out list and the Open risks bullet on in-flight runs": I settled it without a separate finding.
  Scope's Out list and the in-flight bullet both sit in `decision-record.md` already and carry their constraints there.
  The one plan-relevant fact found only in `support-log.md`, the expectation that forks explore more by construction, is folded into F1, since it matters only to D8's hand-off.
- "Attack-surface question on D1's read ban": the verdict is that the Open risks and D8 carry the detection but not the bound, so I raised F5 at LOW: the bound exists in D3, and only its statement is missing.
