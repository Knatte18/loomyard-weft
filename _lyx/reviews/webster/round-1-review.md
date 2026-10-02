---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/loomshed/prrework.go:283
    summary: roundFor allocates a new round for a rejection that supersedes a pending one at a blocked PR-Rework, archiving the stopped session's partial plan as a prior generation and stranding the real retired generation in a classless round
  - id: F2
    severity: MEDIUM
    location: internal/loomshed/prrework.go:291
    summary: a torn or undecodable working-tree record.json is read as belonging to another rejection, so the next Call opens a new round and archives the session's new plan as a prior generation; record.json is also written non-atomically and any read error other than not-exist counts as unfinished
  - id: F3
    severity: LOW
    location: internal/loomshed/prrework.go:174
    summary: a resume after the class write re-runs the whole rework LLM session although its plan already passed the gate and the round only needs its commit
  - id: F4
    severity: LOW
    location: internal/loomshed/generation.go:87
    summary: PlanReviewSkippable classifies only the committed plan, so on Plan-Burler to Plan-Bouncer re-entry the skip can approve and commit uncommitted Burler edits that made a card code-touching
  - id: F5
    severity: LOW
    location: internal/planparser/parse.go:98
    summary: an empty first_card value decodes as a null scalar with empty text, so FirstCardInvalid stays empty and the key silently reads as 1 with no index-file-mismatch finding
  - id: F6
    severity: LOW
    location: internal/loomshed/generation.go:112
    summary: ArchivedWebsterDirs swallows an unreadable rework directory and the DescribeSpec closure swallows stat errors other than not-exist, silently dropping prior generations' run records from Describe
  - id: F7
    severity: LOW
    location: internal/websterengine/archive.go:128
    summary: the webster archive collision names its way forward as re-run the archive, which no operator verb does, instead of the re-step card 3 names, and refusal-spec's rework archive collision row does not cover the webster entry collision
  - id: F8
    severity: LOW
    location: internal/loomengine/config.go:243
    summary: LoomReworkDir's doc comment still says round content is committed with the appended cards
  - id: F9
    severity: LOW
    location: docs/overview.md:366
    summary: card 7's edit joined the validate-plan --rework sentence and the validate-description sentence onto one line with a double space, breaking the one-sentence-per-line rule
  - id: F10
    severity: LOW
    location: internal/shedadapters/bouncer.go:87
    summary: several new or changed Go comments are hard-wrapped at a fixed column mid-sentence, against CONSTRAINTS.md's Comment Line-Break Convention
---

The review range resolved from `_lyx/shed/rework-generations/status.json` (`product.parent: main`), merge base `9ae5f9405`, covering the sixteen card commits `8e5ffcaf5..2c08d0aa2`.
`go build ./...`, `go vet ./...`, the unit tests of every touched package and the `integration`-tagged tests of `loomshed`, `loomcli`, `websterengine` and `planparser` all pass on HEAD.

### [MEDIUM] A superseding rejection archives the stopped session's partial plan as a prior generation

**Location:** `internal/loomshed/prrework.go:283` (`roundFor`), reached from `Call` at line 152.

**Issue:** `lyx loom reject` explicitly accepts a run blocked at PR-Rework, where "a new rejection replaces the pending one" (`internal/loomcli/reject.go:66`, help text line 141).
By then the first rejection's archive is done: `round-N/record.json` exists, classless, carrying the first rejection's identity, and `_lyx/plan/` holds whatever the stopped session wrote (possibly nothing, when it asked a question at once).
For the new identity, `roundFor` finds no matching record and no record-less round, so it allocates `round-N+1`.
`archive` then moves the stopped session's partial plan into `round-N+1/prior-generation/plan` and tells the new session that directory as `PriorPlanDir`, so the session reads a partial or empty plan as the retired generation and loses its framing, Shared Decisions and `## verify:` content.
The real retired generation stays in `round-N`, whose classless record never counts as committed, and the round commit lands both directories.
Separately, `unfinished` is the highest round with no `record.json` among all rounds, not the highest round overall, so a record-less lower round would be picked over a newer committed one and the live plan archived into an old round.

**Fix:** decide on the highest round only.
A record-less highest round is the interrupted archive.
A highest round whose record carries the pending identity is archived.
A highest round whose record is not a committed round (its identity is not among the committed rejections) is the in-flight round of a superseded rejection: adopt it, rewriting `findings.md` and `record.json` with the pending rejection's identity and the recorded `first_card`, class cleared, and run the session over it.
Otherwise open a new round.

### [MEDIUM] A torn or undecodable round record splits the round

**Location:** `internal/loomshed/prrework.go:291`–`299` (`roundFor`) and `450` (`writeRoundRecord`).

**Issue:** `writeRoundRecord` uses `os.WriteFile`, which is not atomic, and it runs twice per round (the completion marker, then the class rewrite).
A crash mid-write leaves a truncated `record.json`.
`roundFor` treats a record that fails to decode as some other rejection's round, so the next Call allocates a new round and archives again; after a torn class rewrite that moves the session's freshly gated plan into `round-N+1/prior-generation/plan` and re-runs the session over an empty plan directory.
Any `ReadFile` error other than not-exist (a permission fault, say) is likewise read as an interrupted archive.
Card 5 requires that a crash at any point resumes into the same round directory.

**Fix:** write `record.json` through a temp file and a rename, and make `roundFor` return an error naming the path and the way forward when a record exists but cannot be read or decoded, treating only not-exist as unfinished.

### [LOW] A resume after the class write re-runs the rework session

**Location:** `internal/loomshed/prrework.go:158`–`178`.

**Issue:** when the working-tree record already carries a class, the session finished, its gate passed, `coverage.md` was copied and only the round commit failed or was interrupted.
The archived branch still builds the session and calls it; the shuttle's attach probe finds no live run, so a whole new LLM rework session spawns and rewrites a plan that was already done.
The round still converges to one directory and one commit, but at the cost of a full session and with the risk of a different plan than the one classified.

**Fix:** when the read record carries a class, skip the session and go straight to the commit and the rejection removal.

### [LOW] The skip seam ignores uncommitted Burler edits on re-entry

**Location:** `internal/loomshed/generation.go:87` (`PlanReviewSkippable`).

**Issue:** Plan-Burler runs `fix-scope: overlay` and commits nothing; its fixes land only through Plan-Bouncer's commit seam.
If the seam errs on Plan-Bouncer's first call (it then reviews for real) and answers true on the re-entry after a Burler round, the Bouncer approves and commits the Burler's uncommitted edits without a judge, though those edits may have turned a `Prosa` card into an `Edit` card.
The recipe comment's bound, that a skip can at most pass a generation whose live cards are themselves exempt, then does not hold for the cards actually committed.

**Fix:** also require `ReviewExempt` over the working-tree plan, so a skip needs both the committed and the working-tree cards exempt; a working-tree parse failure is an error and falls back to a real review.

### [LOW] An empty `first_card:` silently reads as 1

**Location:** `internal/planparser/parse.go:98` (`interpretFirstCard`).

**Issue:** yaml.v3 decodes `first_card:` with no value as a scalar node tagged `!!null` with empty text (checked against the module's yaml.v3).
`Atoi("")` fails, so the value is rejected, but the returned raw text is empty, and `FirstCardInvalid == ""` is exactly the valid-or-absent marker, so no `index-file-mismatch` finding is reported.
Card 1 requires a non-integer `first_card` to reach validation as a finding.

**Fix:** report a non-empty placeholder for an empty value, for example `<empty>`, the way the non-scalar case reports `<non-scalar>`.

### [LOW] Prior run records are dropped silently on an I/O error

**Location:** `internal/loomshed/generation.go:112` (`ArchivedWebsterDirs`) and `internal/loomcli/wiring.go:514`–`518`.

**Issue:** card 11 has Describe told every generation's record.
`ArchivedWebsterDirs` returns nil when the rework directory cannot be read, and the `DescribeSpec` closure treats every `os.Stat` error as absence, so a permission or I/O fault silently drops every prior generation's manual-check items from the PR description.
Skipping a round whose summary does not exist is what card 11 specifies and is not part of this finding.

**Fix:** have `ArchivedWebsterDirs` return its error (an absent directory still lists none), and have the closure return an error for a stat failure other than not-exist, so the Describe spec build fails loudly.

### [LOW] Webster archive collision names a way forward no verb offers

**Location:** `internal/websterengine/archive.go:128`; `contracts/specs/refusal-spec.md:153`.

**Issue:** card 3 names the collision's way forward as "remove whichever copy is stale, then re-step".
The message says "then re-run the archive", which no operator verb does; the archive runs only inside PR-Rework.
The refusal-spec row added for the rework archive collision names only a plan entry or review run directory, not a webster run-record entry, though PR-Rework surfaces both.

**Fix:** reword the message's way forward to "then re-step", and widen the refusal-spec row's trigger to include a webster run-record entry.

### [LOW] Stale doc comment describes appended rework cards

**Location:** `internal/loomengine/config.go:243`.

**Issue:** `LoomReworkDir`'s doc comment says round content is "committed with the appended cards"; cards are no longer appended, and the round now also holds `prior-generation/`.

**Fix:** say it is committed in PR-Rework's round commit with the new plan generation and the archived one.

### [LOW] Two sentences merged onto one line in docs/overview.md

**Location:** `docs/overview.md:366`.

**Issue:** card 7's edit of the `validate-plan --rework` line swallowed the following `validate-description` sentence onto the same line with a double space, against the repo's one-sentence-per-line Markdown rule.

**Fix:** put the `validate-description` sentence back on its own line.

### [LOW] Fixed-column wraps in new and changed Go comments

**Location:** `internal/shedadapters/bouncer.go:87`–`93` and `323`–`325`; `internal/planparser/reviewclass.go:1`–`3`, `9`–`10`, `15`–`18`, `38`–`39`; `internal/websterengine/archive.go:92`–`101`; `internal/planglyph/planglyph.go:34`–`35`; `internal/shedrecipe/entries_bouncer.go:19`–`20` and `129`–`131`; `internal/shedrecipe/recipe.go:135`–`137`; `internal/websterengine/render.go:443`–`444`; `internal/loomcli/wiring.go:467`–`468`; `internal/loomshed/generation.go:48`–`49`; `internal/loomshed/prrework.go:1`–`3`; and in tests `internal/shedadapters/bouncer_skip_test.go`, `internal/shedrecipe/entries_bouncer_test.go:653`, `internal/loomrecipe/approveseam_test.go:60`, `internal/loomcli/wiring_test.go:809`, `internal/planparser/parse_test.go:1178`, `internal/planparser/validate_test.go:1902`.

**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires semantic line breaks in new and changed Go comments, one sentence per line with further breaks only at independent-clause boundaries, and calls existing fixed-column wraps legacy rather than a convention to match.
These comments break mid-sentence at a fixed column, matching the legacy shape of their surrounding files.
The long single-line comments in `prrework.go`, `generation.go` and `wiring.go` comply; this is not a line-width finding.

**Fix:** rewrap each listed comment to one sentence per line, touching no unchanged legacy lines.

## Focus verdicts

- **Crash points (card 5).**
  Mid-archive, including partway through `ArchiveWebster`: converges, since each moved entry has left its origin, the recreated empty review run directories hold no entry, and `ArchiveRunRecord` skips entries already moved.
  After `record.json` and before the session: converges without re-moving anything.
  After Done and the class write but before `Commit`: converges to one directory and one commit, but re-runs the session (F3).
  After `Commit` and before `ClearRejection`: converges through `committed[identity]`.
  `roundFor` can pick a round that is not this rejection's interrupted archive, or open a wrong new round (F1, F2).
- **Commit pathspecs (cards 5 and 16).**
  `reworkCommitPathspec` stages every moved-from side: the plan and rework directories always hold files at commit time, and the reviews root and webster directory are included whenever they hold a file now or the round archived one out of them.
  No branch can hand git an empty-match pathspec.
  `rejectionPending` holds the reviews root and the loom durable directory for the whole window from `lyx loom reject` to `ClearRejection`, on every Stuck, Failed, Paused and resume persist, since only PR-Rework clears the record and only after its round commit.
  Not a defect.
- **Skip seam (cards 8–10).**
  `PlanReviewSkippable`'s round selection is the overview's generation-round rule exactly.
  On generation 0 Plan-Write's own commit lands the plan before Plan-Bouncer runs, so the committed parse succeeds and no Warn is emitted on an ordinary run.
  The one gap found is the Burler re-entry case (F4).
- **ReviewExempt (card 2).**
  Under `go` and absent `language:`, every unparseable, non-self, extensionless and `.go` target counts as required.
  Under `language: none` a target with any extension, `.go` included, is exempt; that is card 2's own definition of a source file under `none`, so it is not raised against the diff.
- **Delete groups (cards 5 and 6).**
  `assert-no-callers` holds: `internal/planparser/appendonly.go`, its test and `internal/websterengine/fingerprint_rebaseline_test.go` are gone, and no reference to `CheckAppendOnly`, `appendOnlyViolation`, `stuckReasonParse` or `RebaselinePlanFingerprint` remains outside `_lyx/`.
  The one stale prose site is F8.
- **Describe records (card 11).**
  Dropping a round whose summary file does not exist is what card 11 specifies; the I/O-error swallowing is F6.
- **Comment convention.**
  See F10.
- **parity_test.go in card 7's commit.**
  Required by card 7's intent: its old `BuiltCardSkipped` case asserted Done for a plan whose committed card would now be checked and fail on create-already-exists, so the gate-parity test had to move to the whole-plan semantics.
  Not a defect.
