---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/01-creel-suite-file.md:37
    summary: The board note slug creel-sandbox-c13-stacked-rewrite is 33 characters, and lyx board upsert refuses any slug over 32, so the D6 note as prescribed cannot be filed
    fix: In card 1's upsert payload on line 37, replace the slug creel-sandbox-c13-stacked-rewrite with a slug of at most 32 characters, for example creel-c13-stacked-rewrite (25 characters); leave every other payload key, the --body-file body and the rule that this never writes creel-sandbox-suite unchanged.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/02-creel-suite-subcommand.md:44
    summary: The new heading assertion in TestRunSuite_PerSuiteSpec pins each suite file's first line, against the decision record's constraint that suite text is checked by review and not pinned by a test
    fix: In card 2's TestRunSuite_PerSuiteSpec bullet, replace the planned assertion that the scheme contains "# " + strings.TrimSuffix(tt.spec.fileName, ".md") with one that reads tt.spec.fileName from the package directory (os.ReadFile(tt.spec.fileName), the test's working directory being tools/sandbox) and asserts it equals tt.spec.doc, so a spec embedding another suite's file still fails its row without pinning any suite text; keep the sentence saying the existing tt.spec.doc check cannot catch a crossed embed.
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/plan/01-creel-suite-file.md:28
    summary: The removal list drops S9 with its trailing separator but not the separator between S8 and S9, which line 30 then requires gone
    fix: In card 1, change "loses S9 with its trailing `---`" to "loses the `---` after S8, S9 with its trailing `---`", so the removal list matches line 30's S8 Verdict line straight to `## Session log format`.
---

### [BLOCKING] Board note slug exceeds the 32-character limit

**Location:** `_lyx/plan/01-creel-suite-file.md:37`

**Issue:** `lyx board upsert --help` ends with "A slug is at most 32 characters.", formatted from `boardengine.MaxSlugLength` (`internal/boardengine/task.go:291`), and `validateSlugLength` refuses a longer slug with `slug exceeds max length of 32 characters`.
`printf %s creel-sandbox-c13-stacked-rewrite | wc -c` prints 33.
The card's prescribed invocation therefore fails, and D6's acceptance criterion (one note filed through `lyx board`) is not met as written.
The rest of the invocation matches `lyx board upsert --help`: `kind` `note` with exactly one type label (`enhancement`) plus the configured `creel` label, and `--body-file` with no `body` key in the payload.
The card writes only the note entry, never the task's own `creel-sandbox-suite`.

### [LOW] Heading assertion pins suite file text

**Location:** `_lyx/plan/02-creel-suite-subcommand.md:44`

**Issue:** The decision record's Constraints say "The suite files' text is checked by review, not pinned by a new test."
The planned `default run` assertion pins every listed suite file's opening heading (core, reed, shuttle and creel), so a heading reword in any of them fails the test.
The defect it guards against, a spec whose `doc` embeds another suite's file, is reachable without pinning text: comparing the embedded `tt.spec.doc` against the on-disk file named by `tt.spec.fileName` catches a crossed `//go:embed` or a crossed `doc:` field equally.
The fix keeps the same target, so no overlay run applies.

### [NIT] Removal list omits the separator after S8

**Location:** `_lyx/plan/01-creel-suite-file.md:28`

**Issue:** In `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md` S8's Verdict line (line 267) is followed by `---` (line 269), then S9, then S9's own `---` (line 287).
Removing "S9 with its trailing `---`", the fixtures and C1 to C13 leaves line 269's `---` before `## Session log format`, while line 30 of the card says the file goes from S8's Verdict line straight to that heading.
Line 30 states the intent; line 28's list should agree with it.

## Rejected

- Card 2 test economy, `unknown flag exits non-zero`: no existing test reaches a suite case's flag-parse failure; `TestRun_RejectsBadInvocation` covers only a missing `-parent`, `fetch` without `-loomyard` and an unknown subcommand, and its bare temp parent could not tell an ignored parse error from the absent-hub exit. Not a defect.
- Card 2 test economy, stderr assertion in `absent Hub exits non-zero`: `TestRunSuite_HubAbsent` pins `runSuite`'s error text, not that `run` prints it per subcommand; the `t.Error` launch stub pins the acceptance criterion's "no agent launched". Not a defect.
- Card 2 `Edit` member glyphs: `lyx quarry resolve` places `suiteSpec` at `suite.go:54-78` (its doc comment is in its span) and `run` at `main.go:84-267`; the `main.go` and `suite.go` header comments lie outside every member span, but adding either file's self glyph beside `run` or `suiteSpec` is refused by `redundant-file-target`, and the files are already reached through those members. Not a defect.
- Card 2 `Create` glyphs: `creelSandboxSuiteMD` and `creelSuite` are two targets on one card, not two cards, so the granularity rule is met. Not a defect.
- Card 1 fidelity: every D1 item (preamble sections, no tmux exceptions section, creel exceptions verbatim plus the operator-observed bullet, `ref` bullet `S9`, `C1`-`C13`, session log, Notes) and every D3 item (S1 to S8 log, `ref` `S1`-`S8`, Notes pointer, S8 last) is carried; no card adds scope outside the decision record.
- Card order: card 1 creates `SANDBOX-CREEL-SUITE.md` before card 2 embeds it and card 3 links it; card 3's `Uses` names the shuttle launchers it copies, which is sound.
- `## verify:`: `tools/sandbox` and `cmd/lyx` are root-module packages covered by `go test ./...`, neither has tagged tests, and the section follows the stencil's order.
- Card 2 attack surface and reachability: the Intent bounds the subcommand to `runSuite`'s fixed writes; `run -parent <tmp> creel-suite` reaches the absent-hub refusal, and `-bogus` after the token passes the top-level parse, which stops at the positional, and reaches the subcommand's flag-parse exit.
