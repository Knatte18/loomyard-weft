---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/14-report-weeks-and-markdown.md:17
    summary: The Week aggregate carries no weekly figure for arbiter repairs, LLM steps, drive reports, parent decisions or alignment outcomes, though the record's Goal and D12 require per-week repairs and hand steps
    fix: "In card 14's Intent, extend the `Week` field list with `Arbiter` (per verdict word, count and applied count, summed over the week's runs holding the arbiter source), `LLMSteps` (per source, summed over the runs holding that source, `landing` included), `DriveReports`, `Decisions` (per `by`) and `Alignment` (per point and verdict, count and applied), each over the runs holding its source with `Lacking` naming the rest and a `nil` figure when no run holds it; add one `TestCompute_Weeks` row asserting a week with one run holding the arbiter source and one lacking it sums the first alone and counts the second in `Lacking[\"arbiter\"]`, and one asserting the LLM-step sum per source. Rendering them in markdown stays optional, since D12's sections are the four board questions."
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/12-report-repairs-and-rounds.md:36
    summary: Every subdirectory of the reviews root is taken as a segment, but the Bouncer's archiveRunDir leaves timestamped siblings such as plan-20261012T101500Z there, so a re-seeded segment yields bogus segment keys and its earlier rounds and escalations are lost
    fix: "Card 5 adds `plan:internal/shedadapters#ArchivedRunDirSegment` -> `func ArchivedRunDirSegment(dirName string) (string, bool)` (in `archive.go`, beside `archiveRunDir`, which writes the `<segment>-<20060102T150405Z>[<suffix>]` names), returning the segment a timestamped run-directory sibling belongs to, with a `TestReviewRounds` row or an existing archive test row pinning it against `archiveRunDir`'s naming. Card 12 lists it under `**Uses:**` and states that a sibling `ArchivedRunDirSegment` accepts is not a segment of its own: its `ReviewRounds` rounds add to its segment's `Rounds`, while `Outcome` stays the live directory's last round; card 11 adds a sibling's `EscalatedRounds` to the segment's `Escalations` count the same way. Card 12 adds a `TestCompute_RepairsAndRounds` row: a run with `plan` holding one round and `plan-20261012T101500Z` holding two reports `plan` with 3 rounds and no `plan-20261012T101500Z` key."
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/09-gitrepo-landing-reads.md:8
    summary: Cards 9, 11, 12 and 13 list a plain test-file path while their Intent changes an existing test or fake (TestReportReads, the allowedImports allowlist TestToldGeometryInvariant_AllowlistOnly asserts, fakeRepo and its BlameLines), where the template asks for member glyphs
    fix: "Card 8 keeps `//internal/gitrepo/reportreads_integration_test.go` and adds the Create `plan:internal/gitrepo_test#TestReportReads` -> `func TestReportReads(t *testing.T)`; card 9 replaces its `//internal/gitrepo/reportreads_integration_test.go` Edit bullet with `plan:internal/gitrepo_test#TestReportReads`. Card 10 keeps its file Creates and adds `plan:internal/reportengine#allowedImports` -> `var allowedImports = []string`, `plan:internal/reportengine#fakeRepo` -> `type fakeRepo struct` and `plan:internal/reportengine#fakeRepo.BlameLines` -> `func (f *fakeRepo) BlameLines(rev, relPath string, lines []int) (map[int]string, error)`, naming `allowedImports` as the allowlist variable in its Intent. Cards 11, 12 and 13 replace `//internal/reportengine/seam_enforcement_test.go` with `plan:internal/reportengine#allowedImports`; card 13 replaces `//internal/reportengine/fake_test.go` with `plan:internal/reportengine#fakeRepo` and `plan:internal/reportengine#fakeRepo.BlameLines`. Cards 11 and 12 keep their `fake_test.go` path, since they only add builders. This combination passed `lyx loom validate-plan --overlay` with no new blocking finding."
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/10-report-run-selection.md:51
    summary: Two finding branches the engine cards prescribe have no row that reaches them, a tag ArchiveTagSlug cannot parse (card 10) and a landing whose slug no board commit holds (card 13, line 38)
    fix: "Card 10 adds a `TestCompute_SelectsRuns` row: an `archive/x` tag (prefix listed, no slug and sha segments) yields one finding naming the tag, no run and no partial run. Card 13 adds a `TestCompute_Landings` row: an exact-joined landing whose slug no board commit holds is kind `other` with one finding naming the slug, and is not blamed."
  - id: F5
    severity: LOW
    class: design
    location: _lyx/plan/15-report-cli.md:41
    summary: Card 15 prescribes two guards no named test reaches, the RequireDrivableWorktree refusal for a non-drivable checkout and refuseNonPrime's branch for a prime name that cannot be resolved
    fix: "Card 15's integration test adds a step before the pair-worktree step: run from the prime's records-side worktree (`fabricengine.RecordsRepoRoot` of the prime's location), it exits 1 with the `RequireDrivableWorktree` refusal. `cli_test.go` gains a table over `refuseNonPrime` like `internal/orchcli/cli_test.go`'s, as rows of `TestConvergence_RefusesBadFlags` or as a third test with `internal/reportcli: 3` in the budget: the prime's own name passes, another name names the prime, and a non-nil `primeNameErr` is an error that wraps it."
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/13-report-landings.md:26
    summary: The exact join says the trailer walk stops at the first match, but card 8's Log has no early stop besides Limit, so as specified each tag walks its records branch's whole first-parent history
    fix: "In card 13's exact-join bullet, replace walked with no limit and stopping at the first match with: walked in growing pages (`Limit` 64, then doubling, re-reading from the tag) until a commit carrying a trailer is found or the walk returns fewer commits than its `Limit`; the merge row whose trailer sits three trailer-free commits below the tag stays as it is."
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/13-report-landings.md:31
    summary: The untagged-slug time join opens at the newest checkpoint after its newest tag's commit, which leaves undefined the most common in-flight case, a slug with no archive tag at all
    fix: "In card 13's time-join bullet, append: a slug with no archive tag opens at its newest checkpoint; and make the not-yet-torn-down row of `TestCompute_Landings` use a slug with no tag, keeping the reused-slug rows for the tagged case."
  - id: F8
    severity: LOW
    class: design
    location: _lyx/plan/15-report-cli.md:59
    summary: The integration step asserts selection.since equals the UTC date 84 days before today while the CLI reads the wall clock itself, so a run straddling UTC midnight fails
    fix: "In card 15's default-since step, take the UTC date before and after the `RunCLIIn` call and assert `selection.since` equals one of the two dates 84 days before them."
---

### [MEDIUM] Weekly aggregates omit repairs and LLM steps

**Location:** `_lyx/plan/14-report-weeks-and-markdown.md:17`

**Issue:** The decision record's Goal reports "per finished run and per ISO week" what "stopped and repaired runs" and "which steps an LLM took by hand".
D12 puts "per-week aggregates" in the JSON envelope, and D5 says weekly aggregates sum over the runs holding each source.
Card 12 computes `RunRow.Arbiter`, `RunRow.LLMSteps`, `RunRow.DriveReports` and `RunRow.Decisions`, and card 11 computes `RunRow.Alignment`, but card 14's `Week` sums none of them.
`Lacking` counts those sources, yet no figure sums them.
As written, a consumer could read repairs and hand steps per run only, and the per-week half of D8's split is not carried by any card.

### [MEDIUM] Review-segment archive siblings are counted as segments

**Location:** `_lyx/plan/12-report-repairs-and-rounds.md:36`

**Issue:** `shedadapters.archiveRunDir` (`internal/shedadapters/archive.go:60`) renames a settled segment's run directory to `<segment>-<20060102T150405Z>` beside it when the Bouncer re-seeds (`internal/shedadapters/bouncer.go:382`).
`LoomReviewsDir`'s doc says those siblings are durable and committed, so they are in the tag's tree.
Card 12 takes every subdirectory but parent-review as a segment.
A re-seeded run then gets a unique timestamped segment key.
That key becomes part of the report's segment set and reads absent for every other run, which pollutes the "do review rounds per segment fall" section.
The live segment's `Rounds` then also drops the earlier generation's rounds.
Card 11 builds its `Escalations` map from the live subdirectory alone, so an escalation recorded in an archived generation turns a `review-escalation` wake into an `awaiting at <Bouncer>` wake.
D9 counts "the numbered rounds with a review file" per segment, and nothing in the record makes a timestamp a segment.

### [LOW] Test and fake changes listed as file paths, not member glyphs

**Location:** `_lyx/plan/09-gitrepo-landing-reads.md:8`

**Issue:** The plan template (`loom-template-plan.md` line 154) asks a card to name an existing test function, fake or helper it changes as a member glyph, and keeps a plain test-file path only for a file the card adds to without changing an existing member.
- Card 9 changes `TestReportReads`, which card 8 creates.
- Cards 11, 12 and 13 each extend the allowlist that card 10's `TestToldGeometryInvariant_AllowlistOnly` asserts.
- Card 13 changes `fakeRepo` with a `BlameLines` call log.

All four list only the file path.
This is one method gap across cards, so it is filed once.
The fix's handles (a `plan:` Create on the creating card, the same handle under `**Edit:**` on the later cards) were overlaid together with F2's new target.
The combined `validate-plan --overlay` run reported no blocking finding, and the `{}` baseline had none either.

### [LOW] Engine finding branches with no reaching row

**Location:** `_lyx/plan/10-report-run-selection.md:51`

**Issue:** Card 10 makes a tag `ArchiveTagSlug` rejects a finding.
`TagRefs("archive/")` lists such a tag, for example `archive/x`, so the input is reachable, but no `TestCompute_SelectsRuns` row names it.
Card 13 makes a landing whose slug no board commit holds kind `other` with a finding (line 38).
No `TestCompute_Landings` row reaches that branch either; its label rows all hold the entry in some board commit.

### [LOW] Card 15 guards with no reaching test

**Location:** `_lyx/plan/15-report-cli.md:41`

**Issue:** The verb calls `fabricengine.RequireDrivableWorktree` (`internal/fabricengine/fabric.go:159`), which refuses a checkout that is not a drivable worktree.
Card 15's tests run only from a temp dir (flags refused first), from the pair's worktree (drivable, refused by `refuseNonPrime`) and from the prime, so that refusal is never reached.
`refuseNonPrime`'s `primeNameErr` branch is likewise unreached.
`internal/orchcli/cli_test.go:124` pins the same branch for orch.

### [LOW] Exact-join walk cannot stop at the first trailer

**Location:** `_lyx/plan/13-report-landings.md:26`

**Issue:** Card 8's `Log` returns the full slice and offers only `Limit` as a cap.
Card 13 asks for a walk that has no limit and stops at the first match, which that API cannot do.
Implemented literally, every tag reads its whole first-parent records history, which carries every landed run's records, so the cost grows with tags times history.

### [LOW] Untagged time join undefined for a never-tagged slug

**Location:** `_lyx/plan/13-report-landings.md:31`

**Issue:** D10 opens an untagged run's window "after the slug's newest archive tag's commit".
A pair landed but not yet torn down on a slug's first run has no tag.
That is the ordinary in-flight case, and neither the plan text nor its test row says which checkpoint opens that window.

### [LOW] Default-since assertion depends on the wall clock

**Location:** `_lyx/plan/15-report-cli.md:59`

**Issue:** The CLI computes the twelve-week default from the real clock, and the test computes "today" separately.
A run crossing UTC midnight between the two compares two different days, so the test can fail with no defect in the code.
`scribe:testing` calls a test that passes only most of the time a bug.

## Fidelity walk

- D1: card 15 mounts `lyx report convergence` with the bare `report` group and operator audience.
- D2: card 15 holds the prime-only refusal and the geometry, and card 10 adds `reportengine` to `PATTERN-told-geometry.md`'s bound packages.
- D3: card 10 makes one ref snapshot, cards 8 and 9 add the `gitrepo` reads, and the verb writes nothing.
- D4: card 10 covers the run unit, partials, ISO week and batten span; card 15 covers the flags and their refusals.
- D5: cards 1 to 7 add the tolerant parses; cards 10 to 14 implement absent-not-zero, apart from the weekly sums F1 names.
- D6: cards 7 and 11.
- D7: card 1 creates the leaf and the `PATTERN-sole-parsers` section; card 11 covers matching and classes.
- D8: cards 12 and 13, apart from the weekly part (F1).
- D9: cards 4, 5 and 12, apart from F2.
- D10: cards 6 and 13.
- D11: cards 9 and 13.
- D12: cards 14 and 15.
- Constraints: the pattern files and the leaf listing are in cards 1 and 10, the hubforge CLI test is in card 15, and every new package gets a budget row.

The Shared Decision `reroute-entries-as-llm-steps` is licensed.
D6 makes the moved-parent resume an automatic step, and D8 splits by actor, so counting that one reroute as an LLM step too would double-count it.
`wake-mean-without-ready-tasks` is licensed as well.
Read literally, D5 and D6 together leave every week without a wake mean, which would leave the Goal's first board question unanswerable.
The plan still reports the source absent per run.

## Rejected

- Card order: no card uses a symbol a later card creates. Card 13 uses cards 6, 8 and 9 and card 10's walk; card 12 uses card 11's `AccountedChild`, which card 11 builds from card 7.
- Verify coverage: `go test ./...` and `go test -tags integration ./...` cover every targeted package, `tmux` and `llm` are compiled, and the order matches the template.
- Re-sign arrows: no Edit card in 1 to 9 changes a signature, and every ImpactSummary states a caller-level conclusion.
- Attack surface: each tolerant parse in cards 1 to 7 states what it skips and that the live strict reader is unchanged, and card 15 states its read-only bound and `--since` cost.
- Granularity and test economy: `failureClass` and `refuseNonPrime` sit in their surface's card, and the lens-review merges left no duplicated row. `TestSoleParser_NoOtherProductionFileSpellsTheNoteTitle` is a new scan; the existing leaf tests only check imports.
- Test budgets: the raised rows match the current counts plus the new tests (battenshed 109, gitrepo 44, fabricengine 644, shedadapters 154, discussionparser 17), and parentreview's 51 stays under its 54.
- The checkpoint commit always touches `_lyx/shed` (`parentRecordsCommitter` commits `RunsRootRel` only and skips an empty commit), so card 10's path-filtered walk does find it.
