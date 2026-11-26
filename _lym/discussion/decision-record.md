## Goal

A later study of landed tasks can read, for each landed run, its decision record, every review round's findings and fixer reports, the parent's answers, the PR-Review findings, the Describe text as merged and the drift verdict.
The decision record, the Code-Review rounds, the Describe text and the drift records are already tracked and reach the archive tag `archive/<slug>/<sha12>` that teardown pushes; this task keeps the rest, which lives only under `.lym` today:
PR-Review findings files, the parent-review ask record and its answer reports, and the landing fixer reports.
It also writes one index row per landed run, adds a `lym report` verb that lists and extracts the kept records from the archive tags, and states that archive tags are kept forever.

## Scope

In:

- Tracked copies of PR-Review findings files, the parent-review ask record and its reports, and landing fixer reports, written when each is produced, in one new tracked directory.
- An index row per landed run, written at the end of Finalize into the run's tracked records.
- A `lym report landed` verb that lists every archived run and extracts one run's kept records into a directory.
- The retention rule for archive tags, stated in PATTERN.

Out:

- Moving the decision record, Code-Review rounds, the Describe text or the drift records: they are already tracked and reach the archive tag.
- Any shared hub-wide file written per landing, such as a ledger on the board branch.
- Changing what PR-Review or the Describe row does; `pr-modes-no-agent` and `describe-review-and-sync` own that.

## Decisions

### D1 — One tracked directory holds the kept copies, inside the status commit

A new tracked directory under the run's `_lym/`, named by `internal/lymdirs`, holds every copy D2 to D4 write: a PR findings subdirectory, an asks subdirectory, a landing fixer subdirectory and the index row.
It sits outside the reviews root, so `lym report`'s walk of review segments under `loomengine.LoomReviewsDirRel` never reads it as a segment.
It joins the loom status commit's pathspec (`statusCommitPathspec` in `internal/loomcli/wiring.go`) and the run-records set `loomengine.PathsOutsideRunRecords`, so every transition commits and pushes it, and a file written there never counts as a dirty path that stops a halted child's re-step on a new build.

Rationale: a copy under the reviews root would show as a fake review segment in the report verbs, and a copy outside the status commit set would ride only teardown's commit and block the batten binary re-step while uncommitted.

### D2 — PR-Review findings are kept

`lym loom pr-findings` copies the findings file it is given into the PR findings subdirectory, for both a send-back and a fix-here answer, and includes the copy in the commit the verb already makes.
Copies share one sequence across both kinds: each takes the next free number in the subdirectory, and its name carries its kind, so no copy overwrites another.
The existing `.lym/loom/` copy, the friction note and the pull-request record stay as they are.

Rationale: the stencil `contracts/stencils/landing/landing-template-pr-review.md` has the agent write findings under `.scratch/`, which is deleted with the worktree, and a fix-here answer keeps only a count and a hash; every findings file passes through this one verb.

### D3 — The parent-review ask record and its reports are kept

Every write to the live parent-review ask record also writes, inside the same locked update and from the value written, a whole-file copy of the record and a copy of the report of every ask number in the record that has one, into the asks subdirectory.
The copies are written atomically through `internal/state`, so a concurrent status commit never stages a half-written file, and the last write under the lock leaves the record's final state.
The live record stays under `.lym/parent-review/` with its lock.

Rationale: an answer to a PR-Review ask, including the parent's own findings it parses into a report, and an answer to a Code-Review ask the round did not accept, is today only in `.lym` and the creel journal.
Writing the copy in the record's one locked update path keeps it ordered with the record whatever process writes, and keeps a future writer from skipping it.

### D4 — Every landing fixer attempt's report is kept

After each landing fixer session ends, whatever its outcome, the report it left in the run's scratch is copied into the landing fixer subdirectory, named by row and by the attempt number from the row's attempt record.
Bound: a session killed before its end, or a lym process killed before the copy, leaves that attempt's report uncopied.

Rationale: `internal/landingshed` writes one scratch report path per row, which each attempt overwrites, so only a copy per session end keeps every attempt.

### D5 — Finalize writes one index row per landed run as its last step

As the last step of Finalize, after the landing and any later fixer, Finalize writes an index row into the kept directory, through `internal/state`, read tolerantly per `PATTERN-tolerant-records`.
The row holds the slug, the board handle and title, the recipe, the parent branch, the landing commit, the landing time as that commit's committer time, the pull request's URL when one was made, the drift verdict when a Drift row ran, and the worktree-relative path of each kept record the run holds.
The landing commit's message is the Describe text as merged, so the row's landing commit names it, beside the run's `_lym/landing/summary.md`.
The record type, its writer and its one reader live in `internal/shedrun`, beside `shedrun.LandedSizeRecord`, so `internal/landingshed` writes it and `internal/reportengine` reads it through the same package.
A resumed Finalize rewrites the row with the same content, since every field comes from durable records; a run that never lands writes none.

Rationale: the archive tag is already the per-run list of landed work and `reportengine.Open` already enumerates the tags; one small file inside each tag gives the entry's index row per landed task without a shared file that every landing would write and push.

### D6 — `lym report landed` lists and extracts the kept records

A new verb under the existing `lym report` group in `internal/reportcli`, computed in `internal/reportengine`, read-only and run where the other report verbs run.

- Runs are ordered by a tag holding an index row before one without, then by the tagged tip's commit time, newest first; every output line shows the tag and its tip time, so `--tag` can pick between two tags of one run.
- With no argument it prints one line per archived run: slug, tag, landing time and commit from the index row, and which kept records the run holds.
  A run with no index row gets a line derived from the tag's tree and the landings join `lym report` already does, marked as derived.
- With `--slug <slug>` it prints that slug's runs with their full index rows, in the order above.
- With `--slug <slug> --extract <dir>` it writes the first run in that order, or the run `--tag <tag>` names, into `<dir>`, keeping `_lym/`-relative paths.
  It writes the same fixed subtrees for every run, with or without an index row: `_lym/discussion/`, `_lym/plan/`, `_lym/reviews/`, `_lym/landing/`, `_lym/drift/` and the kept directory, each where the tag holds it.
  It refuses a `<dir>` that exists and is not empty, naming the way forward: pass an empty or new directory.
  It refuses a slug with no archive tag, naming the way forward: `lym report landed` lists the slugs it has.

Bound: the verb reads only the archive tags present in the local records repository and writes only under `<dir>`; it never fetches and never writes a ref.
A run archived on another machine shows once the records repository has fetched its tag.

Rationale: the analysis behind this entry could not find records that were in the tags; one verb that lists them and turns them back into files closes that gap for every archived run, older ones included.

### D7 — Archive tags are kept forever

No code deletes an `archive/<slug>/*` tag, locally or on origin, apart from the existing cleanup in `internal/fabricengine/cleanupremotewarp.go` of archive tags an older binary pushed under plain `refs/tags/` in a one-repository hub.
The retention is stated in PATTERN (see `## PATTERN changes`).

Rationale: the operator wants these records kept, and a run's records are small text; no pruning exists today, so the rule pins current behaviour against a future prune.

## Constraints

- Every new path is named only through `internal/lymdirs` and the owning package's path accessors (`PATTERN-lym-files`).
- New records reach origin only through the run's records commits and the background pusher, and reach the archive tag through teardown's commit of uncommitted records, which runs before the tag on every teardown path (`PATTERN-pair-remote`, `PATTERN-run-fsm`); no new push or ref write is added.
- The archive tag name, its one-repository namespace and its push stay owned by `internal/fabricengine` (`PATTERN-fabric-boundary`, `PATTERN-single-repo-hub`); the report reads tags through what `reportengine.Open` already uses.
- The index row is a Go-written record: an unknown key is never refused and it is rewritten via `internal/state` (`PATTERN-tolerant-records`); its one reader is the sole parser of the format (`PATTERN-sole-parsers`).
- The new verb is a cobra subcommand of the `report` group (`PATTERN-cli-cobra`).

## Auto-mode assumptions

- D5: the index row is one file per run inside its archive tag, not a row in one shared hub file.
- D4: landing fixer reports are kept, although the entry names only Code-Review fixer reports.
- D7: records are kept forever, with no size or age bound.

## Open risks

- `pr-modes-no-agent` removes PR-Review's findings path and the parent's PR-Review ask; when it lands, D2's copy and the PR-Review part of D3 go with them, and its plan must drop or keep them knowingly.
- `describe-review-and-sync` adds a Describe reviewer whose findings are a new review artifact; the PATTERN rule below expects it to be kept in the run's tracked records.

## Acceptance criteria

- D1: a file written into the kept directory is staged by the loom status commit and is not reported as a dirty path outside run records.
- D1: `lym report` convergence over a run whose kept directory holds copies lists the same review segments as without them.
- D2: a send-back, then two fix-here answers, then a second send-back each leave a tracked copy of their findings file, with distinct numbers and their kind in the name, in the commit `lym loom pr-findings` makes.
- D3: after an ask is recorded, answered and expired, with an answer and an expiry from two processes, the tracked copy of the ask record equals the live record and the report of every answered ask number has a tracked copy.
- D3: an archived run's tree holding the older `parent-review/round-*` directories is still decoded by `reportengine` as before.
- D4: a landing fixer that runs two attempts in one row, the first failing, leaves two tracked reports named by attempt.
- D2, D3, D4: a kept file written after the run's last records commit is present in the archive tag that teardown makes.
- D5: a landed run's kept directory holds an index row whose landing commit, landing time, parent branch and kept-record paths match the run; a resumed Finalize leaves a byte-identical row; a cancelled run holds none.
- D5: the index row reader accepts a row carrying an unknown key.
- D6: `lym report landed` over a hub with one archived run that has an index row and one without lists both, the second marked derived.
- D6: `--slug <slug>` prints each of the slug's runs with its index row, a tag with an index row before one without.
- D6: `--slug <slug> --extract <dir>` writes the first run's decision record, review rounds and kept directory under `<dir>` at their `_lym/`-relative paths; `--tag` selects another run; a run without an index row extracts the same fixed subtrees.
- D6: `--extract` refuses a non-empty directory and a slug with no archive tag, each refusal naming its way forward, and writes nothing.
- D7: Review: the PATTERN entry states that archive tags are never deleted, and no diff adds a deletion of an `archive/` tag.

## PATTERN changes

- `PATTERN-run-records` — changes: Writing a run's friction note, usage or project record: only its one writer (`internal/friction`, shed's usage writer, `lym loom decision add` or `replace`); findings, ask answers, fixer reports and the index row are tracked; archive tags are never deleted.

## Notes for the plan writer

- `RunRow.readSegments` in `internal/reportengine/rounds.go` is the reviews-root walk D1 keeps the kept directory out of.
- The landing fixer's scratch report path and attempt record are in `internal/landingshed/fixerrun.go` and `fixerattempts.go`.
