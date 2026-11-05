# Card 12 — raddleparser

**Edit:**
- `internal/raddleparser/doc.go`
- `internal/raddleparser#TestMain`
- `internal/raddleparser#Proposal`
- `internal/raddleparser#Part`
- `internal/raddleparser#Finding`
- `internal/raddleparser#BoardEntry`
- `internal/raddleparser#BoardContext`
- `internal/raddleparser#Recheck`
- `internal/raddleparser#Parse`
- `internal/raddleparser#Hash`
- `internal/raddleparser#Check`
- `internal/raddleparser#RecheckBoard`
- `internal/raddleparser#TestParse`
- `internal/raddleparser#TestCheck`
- `internal/raddleparser#TestRecheckBoard`
- `internal/raddleparser#TestLeafInvariant_AllowlistOnly`
- `PATTERN.md`

**Extends:**
- `pattern/PATTERN-sole-parsers.md`

**Uses:**
- `internal/discussionparser#DecisionIDs`
- `internal/discussionparser#SectionBullets`
- `internal/discussionparser#ParsePatternChanges`
- `contracts/specs/raddle-proposal-spec.md`

**Intent:** D5 and D6: the proposal's sole parser and its checks, pure, importing the standard library and `internal/discussionparser` only, which `TestLeafInvariant_AllowlistOnly` pins with `scankit.AssertImportAllowlist(t, "internal/raddleparser", "github.com/Knatte18/loomyard/internal/discussionparser")`.
`Parse` decodes the spec's form with `encoding/json`, refusing an unknown key and a malformed file with an error naming the way forward, the spec's file form;
it finds an unknown key by decoding each object also into a `map[string]json.RawMessage` and comparing its keys against the spec's key lists, naming the unknown key in the error, and never calls `Decoder.DisallowUnknownFields`, which `PATTERN-tolerant-records` (test) bans and whose background reserves strictness by other means for an agent-written contract file such as the proposal; `Proposal` and `Part` carry every key card 10's spec pins.
`Hash` is the hex SHA-256 of the bytes.
`BoardContext` is what the checks read of the board, built by the caller: the Sketch slug, the run's entry slug, the Sketch's module labels (its labels that are not type labels), every board entry as a `BoardEntry` (slug, kind, labels, open or done, `revised`), the slugs the raddle store holds (as Sketch slug or filed slug), the board's type labels, the profiles holding a block per recipe, and `Filing` (true at Raddle-File's run).
`Check` returns one `Finding{Check, Part, Detail}` per broken rule of D6, each naming the part, id, slug or entry:
- `decision-unbuilt`: a decision id of the record (`DecisionIDs`) no part builds; `criterion-uncarried`, `risk-uncarried`: an Acceptance criterion or Open risk bullet (`SectionBullets`) no part carries; a bullet a part names that the record lacks is `criterion-unknown` or `risk-unknown`.
- `pattern-uncarried`, `pattern-shared`: a PATTERN change (`ParsePatternChanges`) no part carries, or two parts carry.
- `criteria-miss-decision`: a part whose carried criteria do not name every decision id it builds, so its record would fail the discussion check.
- `carried-decision-unbuilt`: a carried criterion or risk naming a decision id neither the part nor a part on its `depends_on` path builds.
- `board-unchecked`: an open task or project, other than the run's entry, carrying a Sketch module label and absent from `board_checked`.
- `depends-cycle`; `depends-unknown`: a `depends_on` naming no part, no board entry and no `board_checked` entry; a `board_checked` entry gone from the board passes as landed whatever `Filing` says (D6), and `Filing` true only names it among the dependencies Raddle-File drops (D8).
- `packages-shared`: two parts with no `depends_on` path between them listing a common package.
- `decision-unknown`: a decision id a part builds that the record does not hold; `pattern-unknown`: a PATTERN id a part carries that the record's PATTERN changes do not hold.
- `recipe-invalid`, `profile-unknown`, `type-invalid` (a part's or the project's type not in the vocabulary, or `sketch`), `slug-taken` (on the board or in the store, the run's own entry exempt, which `slug-sketch` judges), `slug-sketch` (equal to the Sketch's), `slug-duplicate`.
- `shape-invalid`: a `tapestry` shape with other than one part or one not building every decision, a `project` or `beam` shape with no project, or another shape with one.
- `cost-missing`: a missing proposal or next-best cost line.
`RecheckBoard` is D8's re-check: `Failures` (findings `revised-moved` for a `board_checked` entry still open with another `revised` stamp, `board-unchecked` as above whenever the entry was created or promoted, `slug-taken`), and `Landed`, the `board_checked` slugs now done or gone from the board, named and never failing it.
`TestParse` (tier0) has rows: a valid proposal, an unknown key, a malformed file.
`TestCheck` (tier0) starts from one valid proposal over a fixture record and context, which passes with no finding, then one row per check, each breaking that rule alone and asserting exactly its finding, among them a PATTERN change no part carries and one two parts carry, a criterion naming a decision built by neither the part nor its `depends_on` path, a `board_checked` missing an open entry sharing a module label, a part building a decision id the record lacks, a part carrying a PATTERN id the record's changes lack, a slug the store holds, a part typed `sketch`, a project with no valid type, a part slug equal to the Sketch's, and a `depends_on` on a `board_checked` entry gone from the board that passes, once with `Filing` false (the gate's and `raddle-approve`'s run) and once with `Filing` true.
`TestRecheckBoard` (tier0) has rows: unchanged (passes); a changed `revised`; an open entry absent from `board_checked` sharing a module label, one created after the read and one a note promoted with its old dates; a slug taken on the board and one in the store; a weighed entry now done and one gone (passes, named in `Landed`).
`testmain_test.go` is untagged, the package's `TestMain` as its siblings have.
`PATTERN.md`'s `PATTERN-sole-parsers` line becomes the declared text, its `(test)` marker and background link kept:

```
- `PATTERN-sole-parsers` — Parsing plan, parts file, discussion, summary, recipe, halt note or raddle proposal, or classifying a plan ref: only its sole parser. (test) — [background](pattern/PATTERN-sole-parsers.md)
```

`pattern/PATTERN-sole-parsers.md` gains a `## Raddle proposal (internal/raddleparser)` section: the sole parser, its import rule, and that the decision record is read through `internal/discussionparser` only.

**ImpactSummary:** A new leaf package and one PATTERN line growing by three words; no existing code imports the package yet.
