# Card 5 — Review-file `prior` and `carries`, and the round-check gate entry

**Create:**
- `plan:internal/burlerengine#PriorStatus` -> `type PriorStatus string`
- `plan:internal/burlerengine#PriorResolved` -> `const PriorResolved PriorStatus = "resolved"`
- `plan:internal/burlerengine#PriorOpen` -> `const PriorOpen PriorStatus = "open"`
- `plan:internal/burlerengine#PriorOutdated` -> `const PriorOutdated PriorStatus = "outdated"`
- `plan:internal/burlerengine#PriorWithdrawn` -> `const PriorWithdrawn PriorStatus = "withdrawn"`
- `plan:internal/burlerengine#PriorEntry` -> `type PriorEntry struct`
- `plan:internal/burlerengine#PriorFinding` -> `type PriorFinding struct`
- `plan:internal/burlerengine#RoundContext` -> `type RoundContext struct`
- `plan:internal/burlerengine#CheckReviewRound` -> `func CheckReviewRound(content []byte, ctx *RoundContext) error`
- `plan:internal/burlerengine#RoundCheckGateEntry` -> `func RoundCheckGateEntry(reviewPath string, ctx *RoundContext) shuttleengine.GateEntry`
- `plan:internal/burlerengine#locationPath` -> `func locationPath(location, worktreeRoot string) string`
- `plan:internal/burlerengine#TestCheckReviewRound` -> `func TestCheckReviewRound(t *testing.T)`

**Edit:**
- `internal/burlerengine#Finding`
- `internal/burlerengine#reviewHeader`
- `internal/burlerengine#ParseCurrentReview`
- `internal/burlerengine#checkReviewKeys`
- `internal/burlerengine#Profile`
- `internal/burlerengine#Profile.validateReview`
- `internal/burlerengine#Profile.reviewTable`
- `internal/burlerengine#TestParseCurrentReview`
- `internal/burlerengine#TestEngine_Review_TableShape`
- `internal/burlerengine#TestEngine_Review_Outcomes`
- `internal/burlerengine/doc.go`

**Uses:**
- `plan:internal/burlerengine#FindingDisposition`
- `internal/burlerengine#ReadParentReport`
- `internal/burlerengine#fakeShuttle`
- `internal/burlerengine#halfScript`
- `internal/burlerengine#newEngineForTest`
- `internal/burlerengine#newEngineTestProfile`

**Intent:**
D3 and D4's checks, judged against a round context alone, so `internal/burlerengine` reads no `internal/shedadapters` format.

Shape (`ParseCurrentReview` keeps the shape checks alone):
- `Finding` gains `Carries string` (yaml `carries`); `reviewHeader` gains `Prior []PriorEntry` (yaml `prior`); `PriorEntry` holds `id`, `status` and `answer`.
- `checkReviewKeys` admits `prior` as a third top-level key, and its refusal text names the three keys.
- So the accept after the gate (`acceptReview`) and Fix's re-parse (`acceptedReview`) accept a review that passed the round checks on the cap, with the findings those checks would have refused.

`RoundContext` fields: `Incremental bool`; `Prior []PriorFinding`, each the round N-1 `Finding`, its `*FindingDisposition` (nil when the fixer gave none) and `Held bool` (a `decision`-class finding of a round that routed decisions); `AddedTitles []string` and `AddedTexts []string` (the added entries D2's block lists, in order); `HasChanges bool` and `Changed []string` (the worktree-relative paths the changes file lists); `ParentReportPath string` (empty when the round holds no parent-review ask); `PriorReviewPath`, `PriorFixerReportPath` and `ChangesPath` (for the block); and `WorktreeRoot string`.
`Profile` gains `Round *RoundContext`; nil is a round with no D2 block.
`Profile.validateReview` copies a non-nil `Round` and sets the copy's `WorktreeRoot` to its `worktreeRoot`, so review, probe and resume judge locations alike.

`locationPath` returns a location's text up to the first `:`, `#` or space, made relative to `worktreeRoot` when absolute and cleaned to forward slashes.

`CheckReviewRound` parses the review with `ParseReview` and refuses, each refusal ending `way forward: rewrite the review file ...` with what to change:
- with a nil context: any `prior` entry or `carries` key;
- a `prior` list that misses, repeats or invents an id of `ctx.Prior`, or a status outside the four;
- `withdrawn` on a finding whose disposition is not `disputed`;
- `open` without `answer` on a disputed finding;
- a held finding with a status other than `resolved` or `open`, or `resolved` whose `answer` contains no title of `ctx.AddedTitles`;
- `carries` links that do not match the `open` entries one to one: a carries value naming a non-open or unknown id, an open id carried twice or not at all;
- a carried finding whose class differs from its earlier finding's, whose severity ranks above it (BLOCKING > MEDIUM > LOW > NIT), or whose location path differs from its earlier finding's when `HasChanges` is false or is a path `Changed` lacks when `HasChanges` is true;
- in an incremental round, a new finding (no `carries`) not at `BLOCKING`, or with `HasChanges` one whose location path `Changed` lacks, unless it is exempt;
- in an incremental round, a new `origin: parent` finding that is not exempt.
A new finding is exempt only when its `origin` is `parent`, `ParentReportPath` is set, its `parent_ids` is non-empty and every id is a finding id of the report `ReadParentReport` reads there.
An unreadable report gives no ids. A carried `origin: parent` finding is judged by the carries rules alone.

`RoundCheckGateEntry` is named `round`, has 3 attempts and `PassOnCap: true`, and its gate reads the file and fails with the check's error and a line naming the file.
`Profile.reviewTable` appends it after `ReviewGateEntry`, built from `p.ReviewPath` and `p.Round`, so every seat table of the review half, cluster and assisted rounds included, judges the consolidated review.
doc.go gains a paragraph on the round checks and their bound: a review that passes on the cap keeps its out-of-bound findings and the fixer works them.
Its sentence naming `verdict` and `findings` as the frontmatter's only top-level keys is rewritten to name `prior` as the third.

Tests, tier0:
- `TestCheckReviewRound`, one table, rows per refusal above, each asserting the refusal's `way forward:` clause, and the passing rows the acceptance criteria name: a carried finding moved to a touched file; with no changes file, a carried finding keeping its path; a new BLOCKING finding on a touched file located `path:12` and one located `path#Sym`; with no changes file a new BLOCKING finding on any file; an exempt `origin: parent` finding on an untouched file at LOW; a carried `origin: parent` finding with no ask; two held findings, one `resolved` quoting a listed title and not carried, the other `open` and carried.
  Refused rows include a new parent finding in a round with no ask, with empty `parent_ids`, and with an id the report lacks.
  `locationPath` is reached through rows whose new BLOCKING finding is located by an absolute path under `WorktreeRoot` naming a touched file, which passes, and by `path some prose` cut at the space, which passes for a touched `path` and is refused for an untouched one.
- `TestParseCurrentReview` gains a row: a review carrying `prior` and `carries` passes the shape checks; the stray-key row still refuses.
- `TestEngine_Review_TableShape` asserts the review table's gate is `review` then `round`, the latter with `PassOnCap`.
- `TestEngine_Review_Outcomes` gains a row: an incremental round context and a review whose new finding is MEDIUM fails the round check on every attempt, is let through on the cap, and the result is accepted with that finding among its `Findings`.

**ImpactSummary:** The review half's gate gains one pass-on-cap entry and the shape parse admits two keys; `ParseCurrentReview`'s callers (`acceptReview`, `acceptedReview`, the fix producer, `lym burler validate-review`) accept what they accepted before plus the new keys.
