---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#D5
    summary: D5 claims every row field comes from durable records, but the landing commit is read live as the parent pair's HeadSHA and stored only in the prime's untracked scratch, so a Finalize resumed after another run landed names another commit and time
    fix: "In D5, name the durable source of the landing commit: Finalize writes the landing commit into the run's tracked records (the kept directory) at the moment it records the landing, right after the parent-side merge and before the parent push, and a resumed Finalize past its merge reads it back from there and never re-reads the parent's head. The landing time stays that commit's committer time. Keep the byte-identical-row criterion and add a case where the parent moved on between the landing and the resume."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md#D4
    summary: One landing fixer session spends several turns that each overwrite the same scratch report, so a copy at session end keeps only the last turn and D4's own two-attempts criterion cannot hold
    fix: "Rewrite D4 so the copy is taken after every fixer turn, in the gate check that judges the turn (before a re-prompt or a halt), not at session end. Correct its rationale: each new session already renames the earlier session's report to a timestamped name, and only turns within one session overwrite the report. Keep the criterion that two attempts in one row, the first failing, leave two tracked reports."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#D4
    summary: The attempt number D4 names copies by repeats within a row, because the attempt record restarts for a new row entry, is removed by the operator's way forward, gives back an escalate's turn and is restored after a failed start, so one copy overwrites another
    fix: "In D4, name each copy so that no two copies collide. Use the row, the row entry from the attempt record, and a sequence the copy itself allocates as the next free number in the landing fixer subdirectory for that row and entry, instead of the attempt record's spent count. State that a copy never replaces an existing file."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md#D3
    summary: "D3 claims atomic writes through internal/state keep a concurrent status commit from staging a half-written file, but fsx.AtomicWriteBytes makes its .tmp-* file in the target directory, nothing ignores it, and D1's status commit stages the kept directory whole"
    fix: "In D1, state that the kept directory is created with a tracked .gitignore that ignores the atomic write's .tmp-* temp names, so a status commit or teardown commit racing a write never stages a temp file, and a writer killed before its rename leaves nothing that is committed. Replace D3's unpinned claim with that rule. Add a D1 acceptance criterion: a .tmp-* file in the kept directory is neither staged by the status commit nor reported as dirty."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#D3
    summary: The live ask record is machine-local under .lym, so after an adopt or takeover the new owner numbers asks from 1 again, and D3's whole-file copy and per-number report copies overwrite the history the tracked copy brought from origin
    fix: "In D3, state that a locked write which finds no live record but a tracked copy in the asks subdirectory first seeds the live record from that copy, so ask numbers continue past the copy's latest ask and the whole-file copy keeps every earlier ask. Add a D3 acceptance criterion: an ask recorded after the live record is deleted, with the tracked copy present, takes the next number and leaves the earlier asks and their report copies intact."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md#D6
    summary: Archive tags are also made for cancelled runs, orphan pairs and a diverged teardown's second tag, but D6 lists every archive tag and gives each one without an index row a derived landing line, so a run that never landed reads as landed
    fix: "In D6, state that a tag with no index row whose loom history holds no Finalize entry, the partial-run classification reportengine already makes, is listed as not landed with that reason and without a landing time or commit. Only a tag with Finalize in its history gets a derived landing line. Correct D5's rationale, which calls the archive tags the list of landed work. Add a D6 criterion: a cancelled run's tag is listed as not landed."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md#D6
    summary: "--extract states the bound 'writes only under <dir>' without saying how it holds against a symlink or gitlink entry in the tag tree, a <dir> that is an empty symlink or a non-directory, a --tag outside archive/<slug>/, or a failure part-way through the write"
    fix: "Extend D6's Bound with four rules. (1) It writes regular files only, and lists the symlink and submodule entries of the tag tree as skipped instead of creating them. (2) It refuses a <dir> that is a symlink or a non-directory, naming the way forward to pass a new or empty real directory. (3) It refuses a --tag that is not an archive/<slug>/ tag of the --slug given, naming the way forward that lym report landed --slug <slug> lists the run's tags. (4) It writes into a new sibling directory and renames it to <dir> once every file is written, so a failure leaves <dir> absent. Add the refusals to D6's refusal criterion."
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#D5
    summary: "D5 puts the row at Finalize's last step, after pushRecords, markTaskDone and dropBeamRecord, so a beam landing's pull request URL is already dropped and the row misses Finalize's own records push. The row's 'any later fixer' names nothing, since every fixer runs before the landing"
    fix: "In D5, replace the last-step wording with an ordering: Finalize writes the index row after the parent-side merge and the landing commit's record, reading the pull request URL before any beam record is dropped, and before pushRecords, so Finalize's own status commit and records push carry it. Drop the words 'and any later fixer'."
  - id: F9
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md#D4
    summary: The fix review that Finalize and Parent-Align run over an over-cap fixer change writes its findings to a scratch report under .lym, and no decision keeps it, though the board entry asks for every review round's findings
    fix: "Extend D4, and the Goal's and Scope's lists, to copy each fix review's report into the landing fixer subdirectory when the review session ends, named by row, row entry and the review round from the attempt record, and never replacing an existing copy. Add a D4 acceptance criterion: a row whose fix review rejects once and then approves leaves two tracked fix review reports."
  - id: F10
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md#D2
    summary: Taking the next free number makes every re-run of pr-findings, including the re-run its failed-commit refusal asks for and a re-run over an already counted set, write a duplicate copy. The rationale's claim that every findings file passes through the verb overlooks the stencil's single-change exception
    fix: "In D2, key each copy to the findings file's content hash, which the verb already computes, and skip the copy when a copy with that hash exists, so a re-run writes nothing new. Reword the rationale to say that every findings set the PR-Review agent reports goes through the verb; a single requested change the stencil sends straight to the fix is no findings set."
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md#D5
    summary: "D5's row records worktree-relative paths of the kept records, while D6's --extract writes _lym/-relative paths, so a row path resolved against <dir> misses the file"
    fix: "In D5, record each kept record's path relative to _lym/, the same base D6's --extract writes under <dir>."
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md#D5
    summary: "D5 says the landing commit's message is the Describe text as merged, but Finalize adds the credit and flaky sections, a GitHub merge titles it with the pull request number, and an already-landed run carries the operator's own merge message"
    fix: "Reword D5's sentence to say that the landing commit's message holds the Describe text as merged, with Finalize's credit and flaky sections, and that a merge the operator made by hand carries that merge's own message."
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md#Constraints
    summary: "The Constraints say new records reach origin only through records commits and the background pusher, but D2's copy rides the pr-findings commit, whose committer pushes synchronously through pairteardown.PushRun"
    fix: "In the Constraints bullet, add that D2's copy reaches origin with the pull request record's own synchronous push, which PATTERN-run-fsm keeps outside the background pusher."
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#D7
    summary: D7 exempts the cleanup in cleanupremotewarp.go as code that deletes archive tags, but PlainArchiveTagsOnOrigin only lists plain-namespace tags and prints a command for the operator, so no code deletes an archive tag
    fix: "In D7, drop the exception: say that no code deletes an archive/<slug>/* tag, locally or on origin. Name the plain-namespace listing in cleanupremotewarp.go only as a way forward that the operator runs by hand."
  - id: F15
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md#PATTERN changes
    summary: "The declared PATTERN-run-records text adds archive-tag retention to an entry whose trigger is writing a run's friction, usage or project record, a subject that does not cover deleting refs. It also drops the entry's (test) marker and background link"
    fix: "Keep PATTERN-run-records' declared text to the records clauses (its one writer; findings, ask answers, fixer reports and the index row are tracked), with its (test) marker and background link kept on the overview line. Declare the retention rule as a change to PATTERN-pair-remote, whose subject covers teardown's archive tags: its current text, followed by '; archive tags are never deleted'. Update D7's pointer to match."
---

### [BLOCKING] D5's landing commit has no durable source

**Location:** `_lym/discussion/decision-record.md` D5

**Issue:** D5 says a resumed Finalize rewrites the index row with the same content "since every field comes from durable records".
That is false for the landing commit and its committer time.
- `Finalize.recordLanding` reads `parentHandle.HeadSHA()`, the parent pair's head at call time (`internal/landingshed/finalize.go:738-753`).
- `RecordLanding` writes the value only to the prime's scratch through `shedrun.RecordLandedCommit` (`internal/loomcli/landingdeps.go:249-256`). That record is under `.lym`, is machine-local and is not in the archive tag.
- A Finalize resumed after the push, once another run has landed on the parent, reads that run's commit.

The byte-identical-row criterion then fails, and D6 lists and extracts the wrong commit.
All four lenses raised this.

### [BLOCKING] D4 copies a fixer report only at session end, but one session spends several turns

**Location:** `_lym/discussion/decision-record.md` D4

**Issue:** `fixerRun.run` starts one seat run whose gate allows `Attempts: turnsLeft`.
A failed gate charges a turn and re-prompts in the same session, and every turn writes the one `<row>-fixer-report.md` (`internal/landingshed/fixerrun.go:105`, `:139-143`, `:327-334`).
A copy taken when the session ends keeps only the last turn's report, so D4's own criterion fails: two attempts in one row, the first failing, should leave two tracked reports.
The rationale is also partly wrong.
`archiveStaleReport` already renames an earlier session's report to a timestamped sibling (`fixerrun.go:129`, `:196-203`), so a report is overwritten only between turns of one session.

### [MEDIUM] D4's attempt numbers repeat within a row

**Location:** `_lym/discussion/decision-record.md` D4

**Issue:** The attempt number D4 names its copies by can repeat within a row:
- `ReadFixerAttempts` returns a zero record for a new row entry (`internal/landingshed/fixerattempts.go:79-81`).
- The halt's way forward tells the operator to remove the record (`fixerattempts.go:59`, `:95`).
- An escalate does `Spent--` (`fixerrun.go:296`).
- A failed start writes the original record back (`fixerrun.go:150`).

Each case reuses a number, so a later copy overwrites an earlier one.

### [MEDIUM] Atomic writes do not keep temp files out of the status commit

**Location:** `_lym/discussion/decision-record.md` D3, with D1

**Issue:** `fsx.AtomicWriteBytes`, which `internal/state` uses, creates `os.CreateTemp(dir, ".tmp-")` in the target's own directory (`internal/fsx/fsx.go:57-72`).
The records worktree ignores no `.tmp-*` name; `git check-ignore` matched nothing.
The status commit stages its directories whole (`internal/loomcli/wiring.go:96-101`).

So a status commit that races the orch's answer process stages a half-written temp file under its temp name, and a writer killed before its rename leaves one that the next transition commits.
D3 states the opposite as fact, with no pin.

### [MEDIUM] A machine move restarts the ask numbering over the tracked history

**Location:** `_lym/discussion/decision-record.md` D3

**Issue:** `ParentReviewFilesOf` puts the live record under `.lym/parent-review/` (`internal/burlerengine/parentreview.go:146-149`), and no path seeds it from tracked state.

On a new owner machine:
1. The live record starts empty.
2. `RecordParentAsk` numbers from 1 again (`parentreview.go:206-209`).
3. D3's whole-file copy then replaces the tracked copy that came from origin.
4. `ask-1`'s report copy replaces the earlier `ask-1` copy.

The asks from before the move are lost from the tracked records.

### [MEDIUM] D6 shows runs that never landed as derived landings

**Location:** `_lym/discussion/decision-record.md` D6

**Issue:** Archive tags are not only made for landed runs. They are also made for:
- a cancelled run's discard (`internal/battencli/cancel.go:152`, `:822`);
- an orphan pair (`PATTERN-pair-remote`);
- the origin tip of a diverged teardown (`internal/fabricengine/archive.go:177-187`).

D6 gives every tag without an index row a "derived" line taken from the tag's tree and the landings join.
It never says how a tag of a run that never landed shows, although `reportengine` already sorts such tags into `PartialRun` (`internal/reportengine/runs.go:114-121`, `:163-166`).
D5's rationale calls the tags "the per-run list of landed work", and they are not.

### [MEDIUM] --extract's write bound has no stated rule

**Location:** `_lym/discussion/decision-record.md` D6

**Issue:** This is an attack-surface finding.
D6 states the bound "writes only under `<dir>`" but says nothing on how it holds against:
- a symlink entry in the tag tree;
- an empty `<dir>` that is itself a symlink, which passes the not-empty refusal;
- a `--tag` that names another slug's tag or a ref outside `archive/<slug>/`;
- a failure part-way through the extract.

The refusal criterion says "writes nothing" only for the two refusals it names.

### [MEDIUM] "Last step of Finalize" falls after the records push and the beam record drop

**Location:** `_lym/discussion/decision-record.md` D5

**Issue:** After the merge, `pushParent` runs `pushRecords`, then `markTaskDone`.
Once the push answers Done, `dropBeamRecord` runs (`internal/landingshed/finalize.go:416-424`, `:859-865`).
A row written last therefore has two problems:
- On a beam landing it reads after the pull request record is dropped, so the row loses the pull request URL.
- It misses Finalize's own commit and push, and reaches origin only through the Friction-Reflect transition or teardown.

Fixers and fix reviews run inside `landUnderLock` before the merge (`finalize.go:379-384`), so "any later fixer" names nothing.

### [MEDIUM] The fix review reports are not kept

**Location:** `_lym/discussion/decision-record.md` D4

**Issue:** `runFixReview` writes `<row>-fix-review.md` under the landing scratch directory, `loomengine.LoomScratchDir`, which is `.lym/loom` (`internal/landingshed/fixreview.go:193-215`, `internal/loomcli/landingdeps.go:126`).
That report holds the findings of the review Finalize and Parent-Align run over an over-cap fixer change.
It lives only in `.lym`, and a later session archives it there under a timestamped name.

The board entry asks for "every Code-Review round's findings and the fixer reports" and for findings to "no longer live only in a scratchpad directory".
The Goal's list of what lives only under `.lym` leaves this report out, and no decision keeps it.

### [LOW] Re-runs of pr-findings write duplicate copies

**Location:** `_lym/discussion/decision-record.md` D2

**Issue:** `answer()` commits on both paths, and its failed-commit refusal asks for a re-run (`internal/loomcli/prfindings.go:204-209`).
A fix-here re-run over a set already counted still reaches `answer` (`prfindings.go:222-228`).
With "the next free number", each such re-run copies the same findings again.

The PR-Review stencil also sends a single requested change straight to the fix (`contracts/stencils/landing/landing-template-pr-review.md:59`).
So "every findings file passes through this one verb" holds only for findings sets.

### [LOW] Row paths and extract paths use different bases

**Location:** `_lym/discussion/decision-record.md` D5

**Issue:** D5 records "the worktree-relative path of each kept record", while D6 extracts "keeping `_lym/`-relative paths".
A reader that resolves a row path against `<dir>` does not find the file.

### [LOW] The landing commit's message is not the Describe text alone

**Location:** `_lym/discussion/decision-record.md` D5

**Issue:** The landing commit's message differs from the Describe text in three ways:
- `landingTexts` adds the flaky section and the credit block (`internal/landingshed/finalize.go:154-156`).
- A GitHub merge uses `pullRequestCommitTitle` (`internal/landingshed/finalize_pr.go:496`).
- A task the operator merged on the remote lands through the already-landed path with the operator's own message (`finalize.go:237-238`).

### [LOW] The Constraints misstate how D2's copy is pushed

**Location:** `_lym/discussion/decision-record.md` Constraints

**Issue:** `pullRequestRecordCommitter` commits and then pushes synchronously through `pairteardown.PushRun` (`internal/loomcli/landingdeps.go:527-545`).
`PATTERN-run-fsm` keeps the pull request record's push synchronous and outside the background pusher.
The Constraints bullet says every new record goes through the background pusher.

### [LOW] D7's exception describes no deletion

**Location:** `_lym/discussion/decision-record.md` D7

**Issue:** `PlainArchiveTagsOnOrigin` only runs `ls-remote` and returns a way forward for the operator (`internal/fabricengine/cleanupremotewarp.go:336-379`).
Its caller only reports the list (`internal/fabriccli/fabric.go:904-911`).
No code deletes an archive tag.
The pruning fetch prunes only remote-tracking refspecs, with `--no-tags` (`internal/gitrepo/pull.go:70-74`).

### [LOW] The retention rule sits in an entry whose subject does not cover it

**Location:** `_lym/discussion/decision-record.md` PATTERN changes

**Issue:** `PATTERN-run-records` opens "Writing a run's friction note, usage record or project record", and its rule names each record's one writer.
Archive tags are teardown's refs, which `PATTERN-pair-remote` covers, so "archive tags are never deleted" sits in an entry whose subject does not cover it.

The declared text also ends without the current line's `(test)` marker and background link (`PATTERN.md:35`).
The overview line it prescribes would drop both.

## Rejected

- The bound-breaker lens found that D3's copy is written inside the closure before `state.UpdateJSON` persists, so a failed persist leaves the copy ahead of the record. Rejected: the failed persist returns an error to its caller, and the next locked write rewrites the copy from the record, so D3's "last write leaves the final state" still holds.
- The interactions lens, D1 × D2 and D1 × D4: a kept file left uncommitted does not trip `pr-findings`' or the landing fixer's dirty checks. No finding: both read the code worktree, where `_lym` is ignored. Recorded here only because the focus asked about it.
- The bound-breaker lens: the teardown constraint, "commit of uncommitted records … before the tag on every teardown path". Held: `commitPendingRecords` stages the structural `_lym` before `tagWeftTipForArchive` (`internal/fabricengine/remove.go:307-335`, `internal/fabricengine/junctionnames.go:26`). Local mode makes no `archive/` tag by design, and the orphan-branch cleanup tags a branch with no worktree left to commit.
- Open risks: neither line needs the parent. `pr-modes-no-agent` and `describe-review-and-sync` are both unstarted and depend on `config-keys-nested`. Q5 already settles keeping PR-Review findings until the first lands. No board note titled `Decision for keep-landed-records:` exists.
