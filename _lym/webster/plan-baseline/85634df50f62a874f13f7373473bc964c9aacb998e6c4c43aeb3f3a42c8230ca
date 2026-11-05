# Card 11 — discussion-part-splice

**Create:**
- `plan:internal/discussionparser#SectionBullets` -> `func SectionBullets(record []byte, heading string) []string`
- `plan:internal/discussionparser#PartSplice` -> `type PartSplice struct`
- `plan:internal/discussionparser#SplicePart` -> `func SplicePart(record []byte, part PartSplice) ([]byte, error)`
- `plan:internal/discussionparser#TestSectionBullets` -> `func TestSectionBullets(t *testing.T)`
- `plan:internal/discussionparser#TestSplicePart` -> `func TestSplicePart(t *testing.T)`

**Extends:**
- `internal/discussionparser/doc.go`

**Uses:**
- `internal/discussionparser#DecisionIDs`
- `internal/discussionparser#CheckDecisionIDs`
- `internal/discussionparser#ParsePatternChanges`
- `internal/discussionparser#ParseAddedDecisions`
- `internal/discussionparser#Validate`

**Intent:** D9 step 2's splice, in the decision record's sole parser (`PATTERN-sole-parsers`), standard library only as its leaf test requires.
`SectionBullets` returns the top-level bullets of the `## <heading>` section, each the bullet's whole text with its continuation lines, in order, outside fenced blocks and outside the `<!-- lym:carry-over … -->` marker block of `## Open risks`; an absent section returns none.
The raddle checks (card 12) match a part's carried risks and criteria against these texts.
`PartSplice` carries `Goal` and `Scope` (text the caller renders from the part's contract and packages), `Decisions` (ids), `OpenRisks` and `Criteria` (bullet texts) and `PatternChanges` (entry ids).
`SplicePart` returns a record holding: `## Goal` and `## Scope` from the splice; `## Decisions` with each listed decision copied whole, heading and id included, in the record's order, then every `### Added after Discussion` entry of the record; `## Constraints` and `## Auto-mode assumptions` copied whole; `## Open risks` and `## Acceptance criteria` holding only the listed bullets; `## PATTERN changes`, present only when the part carries one, holding the listed entries' bullets.
A listed id the record does not hold, or a listed bullet or PATTERN entry it does not hold, is an error naming it.
`TestSectionBullets` (tier0) has rows: plain bullets, a bullet with continuation lines, a bullet inside a fence (skipped), the carry-over block (skipped), an absent heading.
`TestSplicePart` (tier0) splices a fixture record with two decisions, an `Added after Discussion` entry and a PATTERN change, and asserts: the spliced record holds exactly the listed decisions plus the added entry, the listed risks, criteria and PATTERN change; it passes `CheckDecisionIDs` with no finding when its criteria name each listed id, and passes `Validate`'s heading check; an unknown id, bullet or PATTERN id is an error naming it.
`doc.go` names the two functions.

**ImpactSummary:** Two new exported functions in a stdlib-only leaf; no existing reader changes.
