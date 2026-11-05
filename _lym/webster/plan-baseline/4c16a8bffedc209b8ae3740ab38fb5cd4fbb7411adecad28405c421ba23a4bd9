# Card 6 — drift-judgement-parser

**Create:**
- `plan:internal/driftengine#Ruling` -> `type Ruling string`
- `plan:internal/driftengine#RulingMatches` -> `const RulingMatches Ruling = "matches"`
- `plan:internal/driftengine#RulingNoMatch` -> `const RulingNoMatch Ruling = "does not match"`
- `plan:internal/driftengine#RulingUnexpected` -> `const RulingUnexpected Ruling = "unexpected"`
- `plan:internal/driftengine#Judgement` -> `type Judgement struct`
- `plan:internal/driftengine#CellRuling` -> `type CellRuling struct`
- `plan:internal/driftengine#CaseRuling` -> `type CaseRuling struct`
- `plan:internal/driftengine#OpenRangeNote` -> `type OpenRangeNote struct`
- `plan:internal/driftengine#JudgementWant` -> `type JudgementWant struct`
- `plan:internal/driftengine#ParseJudgement` -> `func ParseJudgement(data []byte, want JudgementWant) (Judgement, error)`
- `plan:internal/driftengine#TestParseJudgement` -> `func TestParseJudgement(t *testing.T)`

**Edit:**
- `PATTERN.md`
- `pattern/PATTERN-sole-parsers.md`

**Extends:**
- `internal/driftengine/doc.go`

**Uses:**
- `plan:internal/driftengine#DiffLines`
- `plan:internal/driftengine#DiffCommand`

**Intent:** The sole parser of `_lym/drift/judgement.yaml` (D9), strict like the expectation's, and the PATTERN change the decision record declares.

The file's shape: `spawn_id`; under `lines` mode `cells`, a list of `CellRuling` (`id`, `ruling` `matches` or `does not match`, `reason`) and `open_ranges`, a list of `OpenRangeNote` (`case`, `range`, `checked`); under `command` mode `cases`, a list of `CaseRuling` (`case`, `ruling` `matches`, `does not match` or `unexpected`, `reason`).
`JudgementWant` carries what the spawn handed the judge: `Mode`, `SpawnID`, the cell ids (`lines`), the open ranges covering a judged cell as case and range (`lines`), and whether the compare failed (`command`).

`ParseJudgement` refuses, naming the entry: a `spawn_id` other than `want.SpawnID`; under `lines` a missing, duplicate or unknown cell id, a ruling outside the two, an empty reason, a covering open range `open_ranges` does not list, and a `cases` list; under `command` a ruling outside the three, an empty reason, a repeated case, no case at all when the compare failed, and a `cells` list; and under both an unknown field.
Its message is the findings text the judge's `judgement` gate entry re-prompts with (card 15).

`TestParseJudgement` (tier0) is table-driven over D9's criterion: a valid file in each mode parses; a missing, duplicate and unknown cell id, a missing open range, a missing case ruling and an empty list after a failing compare are refused; a ruling outside the mode's set, an empty reason, a repeated case under `command`, a `cells` list under `command`, a `cases` list under `lines` and an unknown field are refused; a file carrying another spawn id is refused.

`PATTERN.md`'s `PATTERN-sole-parsers` line becomes the declared text, verbatim:

```
- `PATTERN-sole-parsers` — Parsing an on-disk format its background lists, or classifying a plan ref: only its sole parser. (test) — [background](pattern/PATTERN-sole-parsers.md)
```

The background file opens with the list of the formats the entry covers, each with its sole parser: the plan, the parts file, the discussion, the summary, the recipe and the halt note as today, and the drift expectation and judgement; any format another task adds, such as a raddle proposal, joins that list.
It gains a `## Drift expectation and judgement (internal/driftengine)` section: `ParseExpectation` and `ParseJudgement` are the only readers of `expected.yaml` and `judgement.yaml`, consumers read their models, and both files are strict.
The doc.go gains a `# Judgement` section, which describes `judgement.yaml` as a file the judge writes, not a record: Drift-Verify removes it once the rulings are stored in `judged.json` (card 16).

**ImpactSummary:** New parser in driftengine with no caller yet; the PATTERN line and its background file gain the two drift formats.
