## Goal

lym loom decision replace folds a run's record under a lock, cuts the history check at it, and refreshes every stored record

Takes: lym board upsert --record-file from record-file-upsert, named by D3's filed refusal and by D6's parent directive; add's guards, refusals and notice in decision.go; CommitAnchoredPathsWithTrailers, FindTrailerCommit and commitEmptySnapshot in fabricengine; Board.HoldsRaddleFiling and raddleRunFiling
Delivers: lym loom decision replace with its guards, base-hash check, carry-over and added-entry checks, trailer commit and notice; the per-run decision lock shared by add, replace, store and writeCarryOver; recordHistory cut at the newest Lym-Record-Replaced trailer; D3's board copy and filed refusal; the D6 spec, parent directive and overview text
Exports: fabricengine.CommitEmptyWithTrailers, a first-parent trailer-key commit list and an anchored file's first-parent history with commit SHAs and positions; a gitrepo.Repo method that commits one path allowing an empty commit; discussionparser checks for carry-over block equality and the added-entry subset

## Scope

The part changes these packages:

- internal/loomcli (about 900 changed lines)
- internal/fabricengine (about 300 changed lines)
- internal/gitrepo (about 80 changed lines)
- internal/discussionparser (about 150 changed lines)
- contracts/specs (about 15 changed lines)
- contracts/stencils/parent (about 15 changed lines)
- docs (about 10 changed lines)

## Decisions

### D1 — `lym loom decision replace` replaces a run's whole record from a file

`lym loom decision replace [<slug>] --from <file> --title <text> --base-sha256 <hex> --by parent|operator|arbiter` is a new subcommand of the existing `decision` cobra group in `internal/loomcli/decision.go`.

- It takes `add`'s caller guards unchanged: `--by` values, the arbiter-strand check, the sampler rule and the parent check.
  So only the run's parent, its arbiter, the run's sampler strand with `--by operator`, or the operator's unnamed shell replaces.
- It is admitted on every run `add` admits, with `add`'s refusals: Discussion-Write running, a perch-mode raddle run at its Raddle-File row, and a Record-Write run whose record is uncommitted or differs from its committed copy.
  It is also refused on a run whose recipe has a Discussion approval row and that is not yet past it; the way forward names `lym loom decision add`.
  A raddle run whose filing is done is refused in every mode, as D3 says.
  It is also refused on a run standing at any row past its plan's approval and not past its recipe's last Webster row.
  A Merriam strand reads no record, and each fresh Master prompt renders only the record's `Added after Discussion` entries,
  so a fold in that span would take an entry out of the view of every Master spawned later.
  The way forward names `lym loom decision add`, whose notice reaches Merriam inline, and a later `replace` to fold the entry once the run has left its Webster rows.
- `add`, `replace`, `store` and the Bouncer carry-over writer `writeCarryOver` in `internal/loomcli/wiring.go` take one per-run decision lock (new), its path formed through `internal/lock`.
  `add`, `replace` and `store` hold it from the first precondition through the commit and the board copy, and `writeCarryOver` holds it across its write and commit.
  The lock so covers every Go writer of the record past approval.
  A second caller waits for it, so a base check and the write it guards are atomic against a concurrent `add`, `replace` or carry-over write.
- Under that lock it refuses unless the sha256 of the record committed at HEAD equals `--base-sha256`, and the working copy equals that committed copy.
  The refusal names no hash; its way forward is to re-read the record, rebuild the file from it and re-run.
- The file must pass the section check and the decision-id and acceptance check (`discussionparser.Validate`, `discussionparser.CheckDecisionIDs`); citations are not tested, as for `add`.
  Its `<!-- lym:carry-over … -->` blocks must equal the current record's blocks, byte for byte, with none added or dropped, since only Go writes those.
  Each of its `### Added after Discussion` entries must equal, byte for byte, an entry of the current record, so it adds and rewrites none.
  A file that fails any of these is refused, and the record is left as it was.
  The subset refusal's way forward is to re-read the record, copy each added entry the file keeps unchanged, record a new call with `add`, and re-run.
- The file may drop `### Added after Discussion` entries and appended `## PATTERN changes` bullets.
  On a record whose `## Goal` is a fenced board body, as Record-Write renders it, the file may rewrite that body.
- It writes the file's bytes as the record and stages and commits them through `fabricengine.CommitAnchoredPathsWithTrailers`, which already takes caller trailers, carrying D2's trailer.
  That seam makes no commit when nothing is staged, and reports committed false.
  `replace` then lands an empty commit carrying the same trailers, through `fabricengine.CommitEmptyWithTrailers` (new) beside `commitEmptySnapshot`.
  That seam commits the record's path alone, allowing the commit to be empty, through a new `gitrepo.Repo` method, and leaves any other staged content staged.
  With git 2.53.0, `git commit --allow-empty -m <msg> -- <record path>`, run while another path is staged and the record is unchanged,
  lands an empty commit and leaves the other path staged.
  So a replace whose bytes HEAD already holds, from a sweep or an earlier commit, still leaves its trailer commit.
- When a commit returns an error, `replace` reads HEAD.
  When HEAD holds the written bytes in a commit that carries this call's trailer, the call succeeds and reports the error as `commit_error` in the envelope, as `add` does.
  When HEAD holds the written bytes without that trailer, it lands the empty trailer commit as the bullet above says;
  when that commit fails too, it refuses with the way forward to re-read the record, rebuild the file from it and re-run `replace`, which lands the trailer commit.
  Only when HEAD lacks the written bytes does it restore the prior bytes and refuse; the way forward is to re-run `replace`, never `lym fabric commit`.
- It takes no `--amends-plan`: a call that needs an approved plan amended is recorded with `add --amends-plan`, and folded by a later `replace` once the run has left its Webster rows.
- It refreshes the board copy as D3 says, and posts each live strand of the run a notice through creel, as `add` does.
  The notice opens with `Parent decision record replaced with lym loom decision replace` (new), names the title and tells the agent to re-read the record whole.
  It must fit creel's one-line field, as `add`'s does.

Bound: replace can drop added entries that the history check otherwise holds to the record, and appended PATTERN bullets;
it cannot rewrite or insert an added entry.
What bounds it is the caller guard, the lock with the base hash, the section and decision-id checks, the kept carry-over blocks,
the added-entry subset check and git history, which keeps every earlier version.
A replace that changes any text outside added entries and carry-over blocks, a dropped appended PATTERN bullet included,
changes `discussionparser.IdentityText` of the record,
so a later Bouncer re-entry whose subject holds the record reviews it again;
a replace that only drops added entries changes no reviewed text.
The Webster refusal reads the row the run stands at when the fold is made.
A fold made after the last Webster row, followed by `lym loom goto --to` a Webster row,
the way forward a Drift-Verify Stuck names as a fix through Webster,
starts a fresh Master whose prompt renders only `Added after Discussion` entries, and the folded call is then absent from it.

Rationale: one whole-record verb covers edit, remove and fold in one checked write.

### D2 — The history check reads versions back to the newest replace only

The replace commit's message carries the trailer `Lym-Record-Replaced:` (new),
followed by the `--by` value, the record's anchor-relative path and the sha256 of the bytes it wrote, separated by spaces.
The record history the dropped-entry check reads (`recordHistory` in `internal/loomcli/projectrecord.go`) finds the newest commit on the records branch whose trailer names this record's path.
It returns only the versions from the cut onward:
the newest version whose bytes hash to that trailer's sha256 and that was committed at or before the trailer commit.
The order is a commit's first-parent position on the records branch, never committer time:
commits made in the same second tie, and the existing path walk, `gitrepo.Repo.PathRevisions`, orders by committer time.
The file-history seam walks the records branch first-parent, as the trailer-list seam does,
and returns each version with its commit SHA and its first-parent position;
the trailer-list seam returns each commit's first-parent position too.
`internal/loomcli` keeps a version when its position is at or before the trailer commit's,
so an empty trailer commit, which is no version of the file, still orders against every version.
A version committed after the trailer commit is never the cut, so a sweep that commits the replaced bytes again after a dropped entry leaves the drop in view.
The read checks each trailer's path itself:
a pair's weft inherits its parent's commits, so a member can carry a beam's trailer for another record.
A commit that swept the replaced bytes before the replace's own commit is therefore the cut point, whichever commit landed them first.
With no trailer naming the record's path, the history is read whole through today's path walk, unchanged,
and so it is when the trailer's sha256 matches no version of the record at or before the trailer commit;
only a cut history is read through the first-parent file-history seam.
The read is split at the fabric boundary.
`internal/fabricengine` gains two general seams (new):
every first-parent commit of the records branch that carries a given trailer key, with its value and first-parent position, newest first;
and an anchored file's first-parent history with each version's commit SHA and first-parent position.
`internal/loomcli` owns the `Lym-Record-Replaced` key, parses its `<by> <path> <sha256>` value and makes the cut in `recordHistory`.
`discussionparser.CheckAddedDecisionsKept` is unchanged; it is handed fewer versions.
Every caller of `recordHistory` reads the bounded history.

Bound: an entry recorded before the newest replace and absent from it is no longer a finding.
The trailer guards against a mistaken writer, not a forged commit:
no shell guard refuses a raw `git commit` in `_lym`, and such a commit can carry the trailer.
The replace that dropped the entry stays in git history with the caller's `--by`.

Rationale: the cut is keyed on the bytes the verb wrote, so it holds whichever commit carried them.

### D3 — `add` and `replace` refresh the board copy of every stored record

`add` and `replace` copy the committed record onto the run's board entry when the run is past its Discussion approval, as today,
or, on a run whose recipe has no Discussion approval row, when the run's board entry carries a stored record.
The second case covers a raddle run of a promoted Sketch and a darn part filed by raddle, whose record Record-Write seeded from the board.
It applies to a run whose record is the discussion record.
It excludes a run whose record is a beam's project record, a recipe with a Beam-Await row such as pawl,
because that record is seeded from the entry's body, while a family project's stored record is the family filing's own.
A run of a recipe with an approval row keeps the past-approval gate alone, so a run sent back into its Discussion segment copies nothing.
`lym loom decision store` is admitted on the same runs, with its caller check unchanged, and refused alike on a run whose filing is done.
The copy goes through `boardengine.Board.StoreDecisionRecord`, which already bypasses the run lock for the record and flag alone.
A failed copy stays a `record_store_error` in the envelope, never a refusal, and its way forward is `lym loom decision store`.
Once the run's raddle filing is done, `add`, `replace` and `store` are refused on that run in every mode, at any row.
The filing counts as done when `raddleRunFiling` reports it, when the run's entry is no longer a task,
or when the board already holds the filing as `boardengine.Board.HoldsRaddleFiling` judges it, given the part slugs of the run's raddle proposal.
So a filing that saved `board.json` and failed to write the raddle store's done marker still refuses;
that refusal's way forward names `lym loom resume` on the run first, which finishes the filing and writes the marker.
This extends the filed refusal `perchFilingRefusal` makes for a perch run.
Its way forward names `add` or `replace` on the filed part a call concerns, or `lym board upsert --record-file` on the family's project for a call that concerns the whole family.
A family filing turns the run's entry into the project and writes the project's own record, so the refusal keeps a copy from overwriting it.

Rationale: without it, a design call on a raddle run leaves the Sketch's stored record stale, and a cancelled raddle run loses it.

### D6 — Specs and the parent directive say when to replace and when to add

`contracts/specs/loom-discussion-spec.md`'s "Parts no writer authors" keeps that a writer or fixer never edits or removes an added entry or appended bullet.
It adds that `lym loom decision replace` may remove an added entry, never edit one, and may drop an appended bullet,
and that the dropped-entry check reads versions back to the newest replace.
`contracts/stencils/parent/parent-directive-parent.md` tells the parent four things:
- record a new call with `add`;
- change or withdraw an earlier decision on a run past its approval with `replace`, folding every added entry into the main decisions;
  on a record whose `## Goal` is a fenced board body, as Record-Write renders it, the fold rewrites that body to carry the call,
  since darn and drift give precedence over the body to added entries alone and a folded D-entry would lose it;
- once a motif run has made a Sketch whose record holds added entries, fold them with `lym board upsert --record-file`;
- before sending a run back to a Webster row after a fold, record each folded call again with `lym loom decision add`.

It also treats a `[creel]` line from the hub's LOOM program that opens with D1's replace notice opening as the parent's instruction, as it does for `add`'s opening.
`docs/overview.md` names the replace verb and the `--record-file` flag where it names `add` and `store`.

This departs from issue #114's first expected point: no fixer folds.
An entry added in Discussion-Bouncer's settling window, after the last fixer round, would be removed without being folded, so the fold is the parent's explicit write.

Rationale: the verbs alone do not keep records current; the parent has to be told to fold.

## Constraints

- `PATTERN-run-records`: the project record's writer list grows by `replace`; see `## PATTERN changes`.
- `PATTERN-refusal-way-forward`: every new refusal of D1 and D3 names its way forward and lands with a test that reaches it.
  D4's refusals in `internal/boardcli`, which the entry does not bind, name their way forward too.
- `PATTERN-fabric-boundary` and `PATTERN-gitrepo-client-boundary`: D1's commit is a call into `internal/fabricengine`.
  D2's history read takes only general fabric seams, a trailer-key commit list and a file history with commit SHAs, both with first-parent positions;
  the `Lym-Record-Replaced` key and the cut stay in `internal/loomcli`.
  D4's live-weft refusal takes one general exported `fabricengine` function (new) reporting whether a slug's weft branch is live;
  `internal/boardcli` runs no git.
- `PATTERN-lym-files`: D1's per-run decision lock path is formed through `internal/lock`.
- `PATTERN-sole-parsers`: every record check is in `internal/discussionparser`, and `raddleparser` keeps reading the record through it alone.
- `PATTERN-tolerant-records`: `record_body_sha256` is a new key of `board.json`, read tolerantly.

## Auto-mode assumptions

- The parent can hash the record file it read to pass `--base-sha256`.
- Before Discussion approval, the Discussion's own review folds added calls into the main decisions. Their appended entries are removed later by a parent `replace` or by `--record-file` on the Sketch.

## Open risks

- A replace during a live agent session, such as Plan-Write or Raddle-Write, reaches the agent only through the creel notice, as `add` does.
  A Bouncer's settling window counts only added-entry titles, so a replace made while a review round is open does not count as a decision made since that round.
- A replace after a perch's last approving round reaches every part's record without a review of the split's seams, the same accepted gap the board's `discussed-task-split` note records for `add`.
- When D3's board copy fails and the run then re-enters its `Motif` block, the seed rewrites the run's record from the stale board copy. `lym loom decision store`, which is the way forward from the `record_store_error`, prevents this.
- The run report counts post-Discussion decisions per `--by` author from the added entries a record still holds, so a folded call drops out of its count.
- D1's Webster refusal reads the row at the fold, so a fold after the last Webster row followed by `lym loom goto --to` a Webster row
  leaves the folded call absent from every Master spawned there.
  D6's parent directive has the parent record the call again with `add` before a goto it makes;
  a goto the operator makes after a Stuck halt is not covered, and the call stays out of Master's view.
- D4 refuses `--record-file` while the weft is live, but `lym board promote` is not refused,
  so a raddle run started in that window adopts the weft and keeps the motif run's committed record, which Record-Write never overwrites.
  The board's stored record does not reach that run, and D3's later copy writes that adopted record over the stored one.
  The window closes when the pair's teardown finishes, and D5's warning does not cover it.

## Acceptance criteria

- D1: a replace whose `--base-sha256` matches the committed record writes the file as the record, commits it with the trailer, and the record at HEAD equals the file.
- D1: a replace whose base differs is refused with a way forward, the refusal names no hash, and the record and HEAD are unchanged.
- D1: a replace on a run whose recipe has a Discussion approval row and that is not past it is refused, naming `lym loom decision add`.
- D1: a file that fails the section check or the decision-id and acceptance check, that drops, changes or adds a carry-over block,
  or that rewrites or inserts an `### Added after Discussion` entry, is refused and the record is unchanged.
- D1: a task session that is not the run's parent is refused, and the parent, the run's arbiter, the run's sampler strand with `--by operator` and the operator's unnamed shell are admitted, as for `add`.
- D1: a replace whose bytes HEAD already holds still leaves a commit carrying its trailer at HEAD, also while another path is staged in the weft index, which stays staged.
- D1: a replace on a run standing at one of its Webster rows, or at a row between its plan's approval and its first Webster row, is refused, naming `lym loom decision add`.
- D1: a replace whose commit fails while HEAD lacks the written bytes restores the prior record and refuses with a way forward that names `replace`.
- D1: a replace whose commit returns an error after HEAD already holds the written bytes succeeds:
  with the trailer at HEAD it reports `commit_error`, and without it it lands the trailer commit.
- D1: an `add` started while a `replace` holds the decision lock runs after it, and its entry is in the record at HEAD.
- D1: a carry-over write started while a `replace` holds the decision lock runs after it, and its block is in the record at HEAD.
- D1: each live strand of the run gets a notice that opens with the replace notice's opening and names the title.
- D2: after a replace that drops an `### Added after Discussion` entry, `lym loom validate-discussion` and the discussion gate report no `added-decision-dropped` finding.
- D2: when the replaced bytes landed in an earlier commit than the trailer commit, the check still reports no `added-decision-dropped` finding for the dropped entry.
- D2: an entry added after a replace and then dropped by a writer is still an `added-decision-dropped` finding.
- D2: when a replace writes X, an `add` makes Y, a writer restores X and a sweep commits it, the check still reports `added-decision-dropped` for Y's entry.
- D2: when HEAD already holds X from a sweep, a replace lands an empty trailer commit, an `add` makes Y, a writer restores X and a sweep commits it,
  the check still reports `added-decision-dropped` for Y's entry.
- D2: a trailer that names another record's path does not bound this record's history.
- D3: `add` and `replace` on a raddle run of a promoted Sketch refresh the entry's stored record, and `lym loom decision store` is admitted on that run.
- D3: `add`, `replace` and `store` on a non-perch raddle run whose family filing is done are refused, naming the filed part and `--record-file` on the project, and the project's stored record is unchanged.
- D3: the same holds for a family filing that saved `board.json` and failed to write the raddle store.
- D3: `add` on a run re-entered into its Discussion segment, whose entry carries a stored record, leaves the stored record unchanged.
- D3: `add` on a beam run leaves the project entry's stored record unchanged.
- Review: D6 — the discussion spec, the parent directive and the overview name `replace` and `--record-file`, when to use each rather than `add`, and the history reset, and the parent directive names the replace notice's opening
  and says a fold on a record whose `## Goal` is a fenced board body rewrites that body.
- Review: D6 — the parent directive says to record each folded call again with `lym loom decision add` before sending a run back to a Webster row after a fold.

## PATTERN changes

- `PATTERN-run-records` — changes: Writing a run's friction note, usage record or project record: only its one writer, `internal/friction`, shed's usage writer or `lym loom decision add` or `replace`. (test)
