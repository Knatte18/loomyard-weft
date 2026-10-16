# Card 5 — discussion-pattern-changes

**Create:**
- `plan:internal/discussionparser#PatternChange` -> `type PatternChange struct`
- `plan:internal/discussionparser#PatternChange.Bullet` -> `func (c PatternChange) Bullet() string`
- `plan:internal/discussionparser#ParsePatternChanges` -> `func ParsePatternChanges(record []byte) ([]PatternChange, []Finding)`
- `plan:internal/discussionparser#DeclaredPatternChanges` -> `func DeclaredPatternChanges(recordPath string) ([]string, error)`
- `plan:internal/discussionparser#ErrPatternChangesInvalid` -> `var ErrPatternChangesInvalid = errors.New("decision record's PATTERN changes section is malformed")`
- `plan:internal/discussionparser#patternChangesHeading` -> `const patternChangesHeading = "## PATTERN changes"`
- `plan:internal/discussionparser#checkPatternChangeMalformed` -> `const checkPatternChangeMalformed = "pattern-change-malformed"`
- `plan:internal/discussionparser#TestValidate_PatternChanges` -> `func TestValidate_PatternChanges(t *testing.T)`
- `plan:internal/discussionparser#TestDeclaredPatternChanges` -> `func TestDeclaredPatternChanges(t *testing.T)`

**Edit:**
- `internal/discussionparser#Validate`
- `internal/discussionparser#TestValidate_CleanRecords`
- `internal/discussionparser/doc.go`
- `contracts/specs/loom-discussion-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/discussionparser#fencedLines`
- `internal/discussionparser#writeFixture`
- `internal/discussionparser#allSectionsContent`

**Intent:**
D5.1: the decision record's optional `## PATTERN changes` section, its shape checked by `discussionparser.Validate`, so the Discussion gates and `lyx loom validate-discussion` share the check.

The bullet shape, defined in `loom-discussion-spec.md`:
`` - `PATTERN-<id>` — adds: <new entry text> ``, `` - `PATTERN-<id>` — changes: <new entry text> `` or `` - `PATTERN-<id>` — removes ``, where `<id>` matches `[a-z0-9-]+`.
A bullet `lyx loom decision add` appends (card 6) ends with the suffix ` — added after Discussion (<by>, <date>)`.

- `PatternChange` holds `ID` (with the `PATTERN-` prefix, no backticks), `Action`, `Text`, and `By` and `Date` (a `time.Time` day), set only on an appended bullet.
  `Bullet` renders the bullet without the suffix, the form every directive and the plan carry.
- `ParsePatternChanges` reads the section between `## PATTERN changes` and the next H2 outside a fenced block, and returns its entries in order and one `checkPatternChangeMalformed` finding per defect: a non-blank line that is not a bullet of the shape, an action other than `adds`, `changes` or `removes`, `adds` or `changes` with empty text, `removes` followed by text, and a second bullet for an entry ID already listed.
  A record without the section returns nil and no finding.
- `Validate` appends `ParsePatternChanges`'s findings, each with `Path` set to the record, after the heading check; a record without the section stays valid, so every in-flight run's record passes.
- `DeclaredPatternChanges` is what each `pattern.Directive` caller reads (cards 9 and 10): an empty `recordPath` or an absent file returns nil and no error; a read error is returned; findings return an error wrapping `ErrPatternChangesInvalid` that names the record path, the section and each finding's detail, and ends with "way forward: the operator fixes and commits the `## PATTERN changes` section of the decision record (the orch never edits the record and escalates this refusal to the operator), confirms it with `lyx loom validate-discussion`, then".
  Each caller completes the sentence with its own re-step.
  Otherwise it returns each entry's `Bullet`, in order.
- `doc.go` names the section, its parse and `DeclaredPatternChanges`.
- `loom-discussion-spec.md` lists `## PATTERN changes` as an optional section placed after `## Acceptance criteria` and before `## Notes for the plan writer`, defines the bullet shape and the suffix above, says the designer lists there every entry its design adds, changes or removes while the PATTERN edit itself lands in a Webster card, and adds to `### Parts no writer authors` that a bullet carrying the suffix is appended only by `lyx loom decision add` and a record that drops one its history held fails the format check wherever the check for dropped `### Added after Discussion` entries runs: the darn discussion gate, `lyx loom validate-discussion` on a darn run and the verb's darn-record check (D5.2's "as for `Added after Discussion` entries"; a loom run's discussion check reads no history for either).

Tests:
`TestValidate_CleanRecords` gains a row whose record carries a well-formed section of each action, an appended bullet included.
`TestValidate_PatternChanges` is table-driven over records built from `allSectionsContent` plus a section: an unknown action, `adds` with no text, `changes` with no text, `removes` followed by text, a stray non-bullet line, an entry ID outside `[a-z0-9-]+` (such as `PATTERN-Foo_bar`) and a repeated entry ID each yield exactly one `pattern-change-malformed` finding naming the line.
`TestDeclaredPatternChanges` covers an empty path, an absent file and a present record with no section (each nil, nil), a clean section (its bullets in order, without the suffix), and a malformed one (an error matching `ErrPatternChangesInvalid` whose text names `lyx loom validate-discussion`).
`cmd/lyx/testdata/test-budget.yaml`'s `internal/discussionparser` row rises by two.

**ImpactSummary:** `Validate` gains one optional-section check; every gate and verb calling it inherits it, and a record without the section is unaffected.
