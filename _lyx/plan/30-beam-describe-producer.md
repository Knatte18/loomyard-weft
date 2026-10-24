# Card 30 — beam-describe-producer

**Create:**
- `internal/landingshed#NewBeamDescribe`
- `internal/landingshed#BeamDescribeDeps`
- `internal/landingshed#TestBeamDescribe`

**Extends:**
- `internal/landingshed/doc.go`

**Uses:**
- `internal/summaryparser#ValidateDescription`
- `internal/summaryparser#Path`

**Intent:**
D11's first beam-recipe row: a Go producer that writes the beam's change description, with no agent.
`BeamDescribeDeps` tells it the description path, the project's title and brief (read through a seam at call time from the board entry of the run's slug), the commit subjects the beam adds over its merge base with the parent (a seam over the task pair's git, oldest first, merge commits left out), and a commit seam that commits the description as Describe's does.
The description is `# <title>` (cut to the 72-character title limit), a blank line, the brief, a blank line and one `- <subject>` line per commit; it is validated with `summaryparser.ValidateDescription`, the same function Describe's gate calls, and a finding is a returned error naming it and, since the landing's claim holds the entry under the run lock, the way out: `lyx batten cancel <slug>` to end the landing, then the title or brief fixed with `lyx board upsert`, then `lyx beam land <slug>` again.
An empty brief with no commit subjects leaves the body empty, which the validation refuses with that way out.
It returns Done once the description is written and committed; a re-run rewrites the same bytes.
`TestBeamDescribe` rows: a title, brief and two subjects give the expected file and Done; an over-long title is cut; an empty brief and no subjects are refused naming `lyx batten cancel <slug>`, `lyx board upsert` and `lyx beam land <slug>`; a read seam error is returned.

**ImpactSummary:** Adds one new producer in landingshed; nothing calls it until the recipe row of card 31.
