## Goal

A later study of landed tasks can read, for each landed run, its decision record, every review round's findings and fixer reports, the parent's answers, the PR-Review findings, the Describe text as merged and the drift verdict.
The decision record, the Code-Review rounds, the Describe text and the drift records are already tracked and reach the archive tag `archive/<slug>/<sha12>` that teardown pushes; this task keeps the rest, which lives only under `.lym` today:
PR-Review findings files, the parent-review ask record and its answer reports, the landing fixer reports and the landing fix review reports.
It also writes one index row per landed run, adds a `lym report` verb that lists and extracts the kept records from the archive tags, and states that archive tags are kept forever.

## Scope

In:

- Tracked copies of PR-Review findings files, the parent-review ask record and its reports, landing fixer reports and landing fix review reports, written when each is produced, in one new tracked directory.
- An index row per landed run, written by Finalize into the run's tracked records before its records push.
- A `lym report landed` verb that lists every archived run and extracts one run's kept records into a directory.
- The retention rule for archive tags, stated in PATTERN.

Out:

- Moving the decision record, Code-Review rounds, the Describe text or the drift records: they are already tracked and reach the archive tag.
- Any shared hub-wide file written per landing, such as a ledger on the board branch.
- Changing what PR-Review or the Describe row does; `pr-modes-no-agent` and `describe-review-and-sync` own that.

## Decisions

### D1 — One tracked directory holds the kept copies, inside the status commit

A new tracked directory under the run's `_lym/`, named by `internal/lymdirs`, holds every file D2 to D5 write: a PR findings subdirectory, an asks subdirectory, a landing fixer subdirectory, the landing record and the index row.
It sits outside the reviews root, so `lym report`'s walk of review segments under `loomengine.LoomReviewsDirRel` never reads it as a segment.
It joins the loom status commit's pathspec (`statusCommitPathspec` in `internal/loomcli/wiring.go`) and the run-records set `loomengine.PathsOutsideRunRecords`, so every transition commits and pushes it, and a file written there never counts as a dirty path that stops a halted child's re-step on a new build.
Every file in the kept directory is written through `internal/state`, whose `fsx.AtomicWriteBytes` gives its temp files `.tmp-*` names in the target's own directory, never through another package's atomic writer.
The records repository's `.git/info/exclude` holds an ignore line for the kept directory's `.tmp-*` names, written only through `internal/fabricengine`.
The line opens with `**/` so it matches under every anchor, since an exclude pattern resolves against each worktree's root and a run's `_lym/` can sit below an anchor subdirectory;
it joins the entries `seedWeftArtifactExcludes` in `internal/fabricengine/recordsgit.go` seeds.
That seeding runs when a pair's junctions are wired, so a new pair has the line before its first write;
it also runs whenever fabricengine takes the records lock directory, which every records commit, the status commit and teardown's commit included, does before it stages.
So an already wired pair and a pair adopted on another machine have the line before any stage of the kept directory, and the exclude file lives in the common gitdir, so one seeding covers every linked records worktree.
So a status commit or teardown commit that races a write never stages a temp file, and a writer killed before its rename leaves nothing that is committed.

Rationale: a copy under the reviews root would show as a fake review segment in the report verbs, and a copy outside the status commit set would ride only teardown's commit and block the batten binary re-step while uncommitted.
The status commit stages the directory whole, so only an ignore rule keeps an atomic write's temp file out of it;
the rule is an exclude line because `PATTERN-fabric-boundary` allows no tracked `.gitignore` on the records side.

### D2 — PR-Review findings are kept

`lym loom pr-findings` copies the findings file it is given into the PR findings subdirectory, for both a send-back and a fix-here answer.
The commit the verb already makes stages the PR findings subdirectory whole, so a copy a failed commit left uncommitted rides the re-run's commit.
Each copy is keyed by the findings file's sha256 content hash, the hash the verb computes for its counted sets, computed on the send-back path too:
a file whose hash a copy already carries is not copied again, so a re-run, including the one the verb's failed-commit refusal asks for, writes nothing new.
A later send-back of a byte-identical file is not copied again either; its first copy holds the same findings.
A new copy takes the next free number in the subdirectory, shared by both kinds, and its name carries its number, its kind and its hash, so no copy overwrites another.
The existing `.lym/loom/` copy, the friction note and the pull-request record stay as they are.

Rationale: the stencil `contracts/stencils/landing/landing-template-pr-review.md` has the agent write findings under `.scratch/`, which is deleted with the worktree, and a fix-here answer keeps only a count and a hash.
Every findings set the PR-Review agent reports goes through this one verb;
a single requested change the stencil sends straight to the fix is no findings set.

### D3 — The parent-review ask record and its reports are kept

Every write to the live parent-review ask record also writes, inside the same locked update and from the value written, a whole-file copy of the record and a copy of the report of every ask number in the record that has one, into the asks subdirectory.
The copies are written atomically through `internal/state`, every report copy before the record copy, so a tracked record copy never names a done ask whose report copy is missing;
D1's exclude line keeps their temp files out of any commit, and the last write under the lock leaves the record's final state.
A read or a write of the record that finds no live record but a tracked copy in the asks subdirectory first seeds the live record from that copy, under the record's lock.
The same seed writes each tracked report copy back to this anchor's live report path, `ParentReviewFiles.ReportPath` of its ask number, and rewrites every seeded ask's `Report` field to that live path,
all under the record's lock before any reader sees the seeded record.
So after an adopt or takeover, every reader sees the asks from before the move and reads each done ask's report, ask numbers continue past the copy's latest ask, and the whole-file copy keeps every earlier ask.
The live record stays under `.lym/parent-review/` with its lock.

Rationale: an answer to a PR-Review ask, including the parent's own findings it parses into a report, and an answer to a Code-Review ask the round did not accept, is today only in `.lym` and the creel journal.
Writing the copy in the record's one locked update path keeps it ordered with the record whatever process writes, and keeps a future writer from skipping it.
The live record is machine-local, so without the seed a new owner would number from 1 again and overwrite the tracked history.

### D4 — Every landing fixer turn's report and every fix review's report are kept

The landing fixer's gate check, which judges each turn of a fixer session, copies the report the turn left in the run's scratch into the landing fixer subdirectory, whatever its verdict, before it re-prompts or halts.
When a fix review session ends, whatever its outcome, the report it left in the run's scratch is copied into the same subdirectory.
Each copy is named by row, by the row entry from the attempt record, by its kind (fixer or fix review), and by a sequence the copy itself allocates as the next free number in the subdirectory for that row, entry and kind.
A copy never replaces an existing file.
Bound: a turn that never reaches the gate check, such as one that times out or is killed, or a lym process killed before the copy, leaves that report uncopied.

Rationale: `internal/landingshed` writes one scratch fixer report path per row, and every turn of one session overwrites it;
each new session already renames the earlier session's report to a timestamped name, so a copy per turn is what keeps every turn.
The fix review that Finalize runs over an over-cap fixer change writes its findings only to scratch.
The attempt record's spent count and review rounds repeat within a row:
a new row entry and the operator's removal of the record reset both, an escalate and a failed start take back a spent turn, and a change past an operator's acceptance resets the review rounds.
So a sequence the copy allocates is what keeps copies apart.

### D5 — Finalize writes one index row per landed run before its records push

Finalize keeps a landing record in the kept directory: the task's kept tip, and the landing commit, written right after the parent-side merge and before the parent push.
Before each merge-in of the parent branch into the task branch, Finalize writes the task branch's tip into the record as the kept tip,
except when the record already holds a kept tip and the task's tip is an ancestor of the parent branch's head, as after a merge-in that fast-forwarded.
So a fixer commit made after a merge-in moves the kept tip, and a fast-forward never does.
The merge-in runs with `--ff` (`Repo.MergeStart` in `internal/gitrepo/merge.go`), so after a hand merge with another run's commit on top it moves the task branch onto that commit;
every search below runs over the kept tip, never over the task branch's tip after a fast-forward.
When the merge makes a commit, the landing commit is the parent's head, and the record is written.
A pull-request merge Finalize makes itself on GitHub records the merge commit GitHub reports for it, before the parent-side merge, which then finds nothing to add.
When the merge reports it was already up to date and the record names a landing commit that fabric shows is an ancestor of the parent branch's head, as on a Finalize resumed after its merge, it reads the commit back from the record and never re-reads the parent's head.
Otherwise the record's landing commit is treated as absent, and the no-record rule below finds the commit and replaces the record.
That covers a hand merge, an adopter's landing, and a parent reset to origin past a discarded local merge.
An up-to-date report does not mean the kept tip is an ancestor of the parent's head:
`squash` defaults to true, and an empty squash also reports up to date with a source that is no ancestor (`TestMergeCrucible_DerivedAlreadyUpToDateIsReadFromTheRecord` in `internal/fabricengine/mergecrucible_integration_test.go`).
The no-record rule, found through fabric, puts ancestry first: the oldest commit on the parent branch's first-parent history that has the kept tip as an ancestor.
When no such commit exists, as after any squash landing, a squash fallback follows: the oldest first-parent commit after the kept tip's merge-base with the parent that holds the task's work by `Fabric.HoldsWork` while its first parent does not.
When the kept tip adds no net code change against that merge-base, or no commit qualifies, the landing record and the index row carry no landing commit and no landing time, and the row says which of the two applies.
A parent the operator reset by hand to a branch that lacks the work gets a new merge and a new record;
one reset to a branch that holds it, as the protected-branch way forward and the adopter-first repair note prescribe, gets that branch's commit by the no-record rule.
The archive of a held parent merge, which resets the parent branch to origin's tip, replaces the record with the landing commit the no-record rule finds on the reset branch.
Finalize then writes the index row into the kept directory once the landing commit is final, after the parent push and any such archive, and before `pushRecords`, so Finalize's own status commit and records push carry it.
It reads the pull request URL before any beam record is dropped.
Both are written through `internal/state` and read tolerantly per `PATTERN-tolerant-records`.
The row holds the slug, the board handle and title, the recipe, the parent branch, the landing commit and the landing time as that commit's committer time or, in their place, the reason the row has none, the pull request's URL when one was made, the drift verdict when a Drift row ran, and the `_lym/`-relative path of each kept record the run holds.
The landing commit's message holds the Describe text as merged, with Finalize's credit and flaky sections, and a merge the operator made by hand carries that merge's own message; the row's landing commit names it, beside the run's `_lym/landing/summary.md`.
The record types, their writers and their one reader live in `internal/shedrun`, beside `shedrun.LandedSizeRecord`, so `internal/landingshed` writes them and `internal/reportengine` reads them through the same package.
A resumed Finalize rewrites the row with the same content, since every field comes from durable records; a run that never lands writes none.

Rationale: `reportengine.Open` already enumerates the archive tags, and a landed run's tag holds its records; one small file inside each tag gives the entry's index row per landed task without a shared file that every landing would write and push.
The parent's head moves on as other runs land, so only the record written at the landing names this run's commit on a resume,
and a landing this Finalize did not make is found in the parent's history from the task's tip as it stood before the merge-in, never at the parent's head.

### D6 — `lym report landed` lists and extracts the kept records

A new verb under the existing `lym report` group in `internal/reportcli`, computed in `internal/reportengine`, read-only and run where the other report verbs run.

- Runs are ordered by a tag holding an index row before one without, then by the tagged tip's commit time, newest first; every output line shows the tag and its tip time, so `--tag` can pick between two tags of one run.
- With no argument it prints one line per archived run: slug, tag, landing time and commit from the index row, or the row's reason when it has none, and which kept records the run holds.
  A tag with no index row whose loom history holds no Finalize entry with the outcome Done is listed as not landed, without a landing time or commit.
  Its reason is the partial run's reason when `reportengine` sorts the tag into a partial run, and otherwise that Finalize never ended Done.
  Only a tag with no index row whose history holds a Finalize entry with the outcome Done gets a line derived from the tag's tree and the landings join `lym report` already does, marked as derived.
- With `--slug <slug>` it prints that slug's runs with their full index rows, in the order above.
- With `--slug <slug> --extract <dir>` it writes the first run in that order, or the run `--tag <tag>` names, into `<dir>`, keeping `_lym/`-relative paths.
  It writes the same fixed subtrees for every run, with or without an index row: `_lym/discussion/`, `_lym/plan/`, `_lym/reviews/`, `_lym/landing/`, `_lym/drift/` and the kept directory, each where the tag holds it.
  It refuses a `<dir>` that exists and is not empty, naming the way forward: pass an empty or new directory.
  It refuses a slug with no archive tag, naming the way forward: `lym report landed` lists the slugs it has.

Bound: the verb reads only the archive tags present in the local records repository and writes only under `<dir>` and a temporary sibling of it, besides removing an empty `<dir>` itself by rule 4;
it never fetches and never writes a ref.
The write bound holds by four rules:

1. It writes regular files only, and lists the symlink and submodule entries of the tag tree as skipped instead of creating them.
2. It refuses a `<dir>` that is a symlink or a non-directory, naming the way forward: pass a new or empty real directory.
   It resolves `<dir>` to an absolute path and refuses the current working directory or one that holds it, naming the way forward: pass a new directory or one outside the current directory.
3. It refuses a `--tag` that is not an `archive/<slug>/` tag of the `--slug` given, naming the way forward: `lym report landed --slug <slug>` lists the run's tags.
4. It writes into a new sibling directory, named after `<dir>` with an extract suffix, which it creates with a create that fails when the path exists.
   It refuses an existing sibling of that name, a symlink included, naming the way forward: remove the sibling by its path.
   Once every file is written, it removes an existing empty `<dir>` with a remove that deletes only an empty directory, right before it renames the sibling onto `<dir>`,
   so the rename never targets an existing directory.
   A failure removes the sibling and leaves `<dir>` as it was, except a failed rename after that removal, which leaves `<dir>` absent;
   a process killed mid-extract leaves the sibling for the operator to remove.

A run archived on another machine shows once the records repository has fetched its tag.

Rationale: the analysis behind this entry could not find records that were in the tags; one verb that lists them and turns them back into files closes that gap for every archived run, older ones included.
Archive tags are also made for cancelled runs, orphan pairs and a diverged teardown's origin tip, so a tag alone does not mean the run landed.

### D7 — Archive tags are kept forever

No code deletes an `archive/<slug>/*` tag, locally or on origin.
The listing of archive tags under origin's plain `refs/tags/` in `internal/fabricengine/cleanupremotewarp.go` deletes nothing: it prints a way forward that the operator runs by hand.
The retention is stated in PATTERN (see `## PATTERN changes`).

Rationale: the operator wants these records kept, and a run's records are small text; no pruning exists today, so the rule pins current behaviour against a future prune.

### Added after Discussion (arbiter, 2026-11-26): Round-3 F20 and F21 are answered by D1 and D5 as written

Decision: D1's ignore line for the kept directory's .tmp-* names in the records repository's .git/info/exclude, seeded by seedWeftArtifactExcludes through mutateGitExclude, stands as written, and so does D5's rule that reads the landing commit back from the landing record only when fabric shows it is an ancestor of the parent branch's head. Review round 3's F21 (tracked .gitignore) and F20 (record trusted after a parent reset) need no further change.

Rationale: seedWeftArtifactExcludes in internal/fabricengine/recordsgit.go writes through fabricengine.mutateGitExclude, which PATTERN-fabric-boundary names as the only exclude mutator. It runs at junction wiring (junction.go) and in ensureWeftLockDirAt, which every records commit takes before it stages, so the line is in place for a new, an already wired and an adopted pair. With the ancestry check, a record naming a local merge discarded by git reset --keep origin/<parent> is treated as absent, and the no-record rule finds the commit on the reset parent.

### Added after Discussion (parent, 2026-11-26): D5 kept tip survives Finalize's repeated merge-ins

Decision: D5 keeps the fixer's rule with one more exception: Finalize never rewrites the kept tip when the task's tip is still the commit the last merge-in left, the merge record's MergeCommit, or its TaskHead when that merge-in was up to date. D5 gains the acceptance criterion: a hand squash of the task into the parent, then a gate halt, then a resume, gives a row naming the squash commit.

Rationale: Option (b) of the hard-cap question: it closes F19's hole on resumes and retry passes and still catches a merge fixer's later commits, which option (a) relies on a parent-side merge commit to name.

## Constraints

- Every new path is named only through `internal/lymdirs` and the owning package's path accessors (`PATTERN-lym-files`).
- New records reach origin only through the run's records commits and the background pusher, except D2's copy, which reaches origin with the pull request record's own synchronous push that `PATTERN-run-fsm` keeps outside the background pusher.
  They reach the archive tag through teardown's commit of uncommitted records, which runs before the tag on every teardown path (`PATTERN-pair-remote`, `PATTERN-run-fsm`); no new push or ref write is added.
- The archive tag name, its one-repository namespace and its push stay owned by `internal/fabricengine` (`PATTERN-fabric-boundary`, `PATTERN-single-repo-hub`); the report reads tags through what `reportengine.Open` already uses.
- The index row and the landing record are Go-written records: an unknown key is never refused and each is rewritten via `internal/state` (`PATTERN-tolerant-records`); their one reader is the sole parser of the format (`PATTERN-sole-parsers`).
- The new verb is a cobra subcommand of the `report` group (`PATTERN-cli-cobra`).

## Auto-mode assumptions

- D5: the index row is one file per run inside its archive tag, not a row in one shared hub file.
- D4: landing fixer and fix review reports are kept, although the entry names only Code-Review fixer reports.
- D7: records are kept forever, with no size or age bound.

## Open risks

- `pr-modes-no-agent` removes PR-Review's findings path and the parent's PR-Review ask; when it lands, D2's copy and the PR-Review part of D3 go with them, and its plan must drop or keep them knowingly.
- `describe-review-and-sync` adds a Describe reviewer whose findings are a new review artifact; the PATTERN rule below expects it to be kept in the run's tracked records.

## Acceptance criteria

- D1: a file written into the kept directory is staged by the loom status commit and is not reported as a dirty path outside run records.
- D1: a `.tmp-*` file in the kept directory is neither staged by the status commit nor reported as dirty.
- D1: `lym report` convergence over a run whose kept directory holds copies lists the same review segments as without them.
- D2: a send-back, then two fix-here answers, then a second send-back, each over a distinct findings file, each leave a tracked copy with distinct numbers and their kind in the name, in the commit `lym loom pr-findings` makes; a re-run over a file already copied leaves no new copy.
- D3: after an ask is recorded, answered and expired, with an answer and an expiry from two processes, the tracked copy of the ask record equals the live record and the report of every answered ask number has a tracked copy.
- D3: an ask recorded after the live record is deleted, with the tracked copy present, takes the next number and leaves the earlier asks and their report copies intact; a read after the same deletion returns the earlier asks.
- D3: after the live record and the live reports are deleted with the tracked copies present, the PR-Review standing read and the round settle of a done ask read its report and succeed.
- D3: a write killed between a report copy and the record copy leaves a tracked record copy whose every done ask has its report copy.
- D3: an archived run's tree holding the older `parent-review/round-*` directories is still decoded by `reportengine` as before.
- D4: a landing fixer that runs two turns in one row, the first failing, leaves two tracked reports with distinct sequence numbers.
- D4: a copy made after the row's attempt record was removed or its turn given back leaves every earlier copy of the row intact.
- D4: a row whose fix review rejects once and then approves leaves two tracked fix review reports.
- D2, D3, D4: a kept file written after the run's last records commit is present in the archive tag that teardown makes.
- D5: a landed run's kept directory holds an index row whose landing commit, landing time, parent branch and kept-record paths match the run, and the row is in the commit Finalize's records push carries; a resumed Finalize leaves a byte-identical row, also when another run landed on the parent between the landing and the resume; a resumed Finalize whose merge makes a new commit records that commit; a cancelled run holds none.
- D5: a parent that received the task's work by a hand merge, with another run's commit landed on top of it, gives an index row naming the hand merge, through a Finalize that merges that parent into the task branch itself.
- D5: a pull-request squash landing Finalize makes gives a row naming the squash commit; so do a hand squash merge and an adopter-first squash with another commit on top.
- D5: a task whose commits leave no net code change gives a row with no landing commit and no landing time, which says so.
- D5: a parent push refused as a protected branch, the work then merged on GitHub by hand and the parent reset to origin, gives a row naming the commit on origin's parent.
- D5: a beam landing's index row holds its pull request URL.
- D5: the index row reader accepts a row carrying an unknown key.
- D6: `lym report landed` over a hub with one archived run that has an index row and one landed run without lists both, the second marked derived.
- D6: a cancelled run's tag is listed as not landed, with its reason, also when its history holds a Stuck Finalize entry.
- D6: `--slug <slug>` prints each of the slug's runs with its index row, a tag with an index row before one without.
- D6: `--slug <slug> --extract <dir>` writes the first run's decision record, review rounds and kept directory under `<dir>` at their `_lym/`-relative paths; `--tag` selects another run; a run without an index row extracts the same fixed subtrees; an extract into an existing empty directory writes the run there; a symlink entry in the tag tree is listed as skipped and not created.
- D6: `--extract` refuses a non-empty directory, a `<dir>` that is a symlink or a non-directory, a slug with no archive tag, a `--tag` outside the slug's `archive/<slug>/` tags and a leftover extract sibling, a symlink included, and the current working directory, each refusal naming its way forward, and writes nothing.
- D6: an extract that fails part-way leaves no sibling behind and `<dir>` as it was, absent or empty, except a rename that fails after an empty `<dir>` was removed, which leaves `<dir>` absent.
- D7: Review: the PATTERN entry states that archive tags are never deleted, and no diff adds a deletion of an `archive/` tag.

## PATTERN changes

- `PATTERN-run-records` — changes: Writing a run's friction note, usage or project record: only its one writer (`internal/friction`, shed's usage writer, `lym loom decision add` or `replace`); findings, ask answers, fixer and fix review reports, the landing record and the index row are tracked. (test) — [background](pattern/PATTERN-run-records.md)
- `PATTERN-pair-remote` — changes: Pushing run records, tearing down a pair or deleting an archive tag: the code branch follows the records side via the background pusher no transition waits for; remote steps hold the slug's teardown lock; teardown only via `internal/pairteardown`; archive tags are never deleted. (test) — [background](pattern/PATTERN-pair-remote.md)

## Notes for the plan writer

- `RunRow.readSegments` in `internal/reportengine/rounds.go` is the reviews-root walk D1 keeps the kept directory out of.
- The landing fixer's scratch report path, its gate check and attempt record are in `internal/landingshed/fixerrun.go` and `fixerattempts.go`; the fix review's report path is in `fixreview.go`.
- The landing commit is read in `Finalize.recordLanding` in `internal/landingshed/finalize.go`, at the merge and again after an archived parent merge;
  the first-parent searches D5 adds belong in `internal/fabricengine`, beside `Fabric.HoldsWork` in `parenttips.go`.
- Finalize's merge-in is `Finalize.mergeParent` in `internal/landingshed/finalize.go`; D5's kept task tip is written there, before the merge-in.
- `pullrequest.Client.Merge` returns only an error; D5's GitHub merge commit needs it to return the SHA GitHub's merge answer carries, recorded in `Finalize.mergeOnGitHub` in `internal/landingshed/finalize_pr.go`.
- `reportengine`'s partial-run classification of a tag is `readLoomRecords` in `internal/reportengine/runs.go`.
