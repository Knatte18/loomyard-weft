```yaml
lens: leanness
outcome: reported
```

Task: lens review, leanness. I looked for content in the decision record that should move or be cut.

**Findings**

1. MEDIUM, D2's table. The rows for the network, GitHub API, push-lock and catch-up classes and for "Stale build" say "as `push-retry-not-halt` settles it" or "as today". They carry no requirement for this task.
   - Evidence: Scope Out already says that task owns those classes, and the Constraints bullet on `push-retry-not-halt` cites them again.
   - Completeness test: one requirement lives in that network row, the "refused credentials" severe verdict. Keep that cell's verdict. Cut the "as that task says" cells or fold them into one line. As written the same fact appears in three places.

2. MEDIUM, D9 restates D6.
   - Evidence: D9's last bullet, "An open BLOCKING finding the fixer did not defer is held, never accepted (D6)", repeats D6's unattended paragraph.
   - D9's "Under `true` the role text is unchanged but for D7's relay rule" repeats D7's role-text sentence.
   - D6's Review criterion and D9's Review criterion both check the hold rule in `role-autonomy.md`.
   - Fix: keep the rule in D6, have D9 point to it, and cut the duplicate criterion clause.

3. LOW, D8's Rationale and D3's Rationale. "open point N of the study" plus a paraphrase of the study is deliberation narrative.
   - The one-line reference to the study may stay. The paraphrase of why ("waits by design that block no other work") is already stated in the decision text. Move the paraphrase to the support log.

4. LOW, D13's Rationale. "the glossary rules out 'full auto'; 'unattended' says the operator is away without tying the level to a time of day" duplicates the support log's rejected alternative "Name the level `overnight`".
   - The justification for the chosen name belongs in the support log. Keep only that the operator left the name open.

5. LOW, Auto-mode assumptions. Two lines restate decisions rather than assumptions.
   - "The stricter level's name is `unattended`" restates D1 and D13.
   - "The bounds in the table are the study's proposed ones" restates D2's table.
   - These are also recorded in the support log's Q1. Cut them, or keep only the reason each was assumed.

6. LOW, Open risks, third bullet. "their counts are not halts at a row, so they never reach the third-halt verdict" is a design statement, not a risk.
   - It belongs as a sentence in D3, next to the list of what counts nothing.
   - Completeness test: it carries a requirement (dead-driver and API-error stops do not count), so move it rather than cut it.

7. LOW, Notes for the plan writer, first bullet. "The study's section 4 names where each lym-settled class is handled today" partly overlaps the Goal's citation of the study.
   - Acceptable as a hint. This is emphasis only.

8. LOW, Goal. The first two sentences ("lym runs overnight…", "Today runs stop…") restate the board entry.
   - The third sentence onward carries the goal. The first two could shrink to one clause.

9. LOW, Scope Out. The "Classes the study shows stopped recurring" bullet lists four classes by name. That is a snapshot of the study.
   - Per the prose rule on perishable specifics, cite the study's "Classes that stopped recurring" table instead of listing its rows.

Other things I noticed (emphasis only):

- D7 and D10 both define how a severe post is made under unattended. D10 already says "by D7's path", which is fine.
- D11 is long, but each bullet carries a requirement, so it should not be cut.

Tool calls: none. My context held both files and the rubric.
