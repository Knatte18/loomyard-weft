# Decision record — a Sketch or decision record can be edited or replaced, and raddle follows the newer design

## Goal

A decision record holds the current design only, the way a board body does.
A run's parent or the operator can replace a run's record in place, folding later calls into the main decisions, with git history keeping the old text.
The operator can replace a Sketch's stored record on the board, so a promoted Sketch's raddle reads, and validates against, the current design.

## Scope

In: D1 to D6 below.

Out:

- Raddle validating against the board body; raddle keeps reading the stored record alone.
- A per-decision `edit <Dn>` or `remove <Dn>` verb.
- Writing a record onto an entry that has none.
- Discussion fixers folding or removing `### Added after Discussion` entries: they keep never touching them.
- Any change to this task's own board entry.

## Decisions

### D1 — `lym loom decision replace` replaces a run's whole record from a file

`lym loom decision replace [<slug>] --from <file> --title <text> --base-sha256 <hex> --by parent|operator|arbiter` is a new subcommand of the existing `decision` cobra group in `internal/loomcli/decision.go`.

- It takes `add`'s caller guards unchanged: `--by` values, the arbiter-strand check, the sampler rule and the parent check, so only the run's parent, its arbiter or the operator's unnamed shell replaces.
- It is admitted on every run `add` admits, with `add`'s refusals: Discussion-Write running, a perch-mode raddle run at its Raddle-File row, and a Record-Write run whose record is uncommitted or differs from its committed copy.
  It is also refused on a run whose recipe has a Discussion approval row and that is not yet past it; the way forward names `lym loom decision add`.
- `add`, `replace` and `store` take one per-run decision lock (new), its path formed through `internal/lock`, held from the first precondition through the commit and the board copy.
  A second caller waits for it, so a base check and the write it guards are atomic against a concurrent `add` or `replace`.
- Under that lock it refuses unless the sha256 of the record committed at HEAD equals `--base-sha256`, and the working copy equals that committed copy.
  The refusal names no hash; its way forward is to re-read the record, rebuild the file from it and re-run.
- The file must pass the section check and the decision-id and acceptance check (`discussionparser.Validate`, `discussionparser.CheckDecisionIDs`); citations are not tested, as for `add`.
  It must hold every `<!-- lym:carry-over … -->` block the current record holds, byte for byte, since only Go writes those.
  A file that fails any of these is refused, and the record is left as it was.
- The file may drop or rewrite `### Added after Discussion` entries and appended `## PATTERN changes` bullets.
- It writes the file's bytes as the record and commits them through `fabricengine.CommitAnchoredPathsWithTrailers`, which already takes caller trailers, carrying D2's trailer.
  The commit is made even when the record's bytes already landed in an earlier commit, so the trailer always exists.
  When the commit fails, it restores the prior bytes and refuses; the way forward is to re-run `replace`, never `lym fabric commit`.
- It takes no `--amends-plan`: a call that needs an approved plan amended is recorded with `add --amends-plan`, and folded by a later `replace` once the amendment round has settled.
- It refreshes the board copy as D3 says, and posts each live strand of the run a notice through creel, as `add` does, naming the title and telling the agent the record was replaced and to re-read it whole.
  The notice must fit creel's one-line field, as `add`'s does.

Bound: replace can drop added entries and appended PATTERN bullets that the history check otherwise holds to the record.
What bounds it is the caller guard, the lock with the base hash, the section and decision-id checks, the kept carry-over blocks and git history, which keeps every earlier version.
A replace changes `discussionparser.IdentityText` of the record, so a later Bouncer re-entry whose subject holds the record reviews it again.

Rationale: one whole-record verb covers edit, remove and fold in one checked write.

### D2 — The history check reads versions back to the newest replace only

The replace commit's message carries the trailer `Lym-Record-Replaced:` followed by the `--by` value and the sha256 of the bytes it wrote (new).
The record history the dropped-entry check reads (`recordHistory` in `internal/loomcli/projectrecord.go`) finds the newest commit on the records branch that carries that trailer for the record, and returns only the versions from the newest one whose bytes hash to the trailer's sha256 onward.
A commit that swept the replaced bytes before the replace's own commit is therefore the cut point, whichever commit landed them first.
With no such trailer, the history is read whole, as today.
The read lives in `internal/fabricengine`, beside `fabricengine.CommittedAnchoredFileHistory`.
`discussionparser.CheckAddedDecisionsKept` is unchanged; it is handed fewer versions.
Every caller of the history seam reads the bounded history.

Bound: an entry recorded before the newest replace and absent from it is no longer a finding.
Only a lym verb writes a commit message on the records branch, and the replace that dropped the entry stays in git history with the caller's `--by`.

Rationale: the cut is keyed on the bytes the verb wrote, so it holds whichever commit carried them.

### D3 — `add` and `replace` refresh the board copy of every stored record

`add` and `replace` copy the committed record onto the run's board entry when the run is past its Discussion approval, as today, or when the run's board entry carries a stored record.
The second case covers a raddle run of a promoted Sketch and a darn part filed by raddle, whose record Record-Write seeded from the board.
`lym loom decision store` is admitted on the same runs, with its caller check unchanged.
The copy goes through `boardengine.Board.StoreDecisionRecord`, which already bypasses the run lock for the record and flag alone.
A failed copy stays a `record_store_error` in the envelope, never a refusal, and its way forward is `lym loom decision store`.

Rationale: without it, a design call on a raddle run leaves the Sketch's stored record stale, and a cancelled raddle run loses it.

### D4 — `lym board upsert --record-file <file>` replaces an entry's stored record

`lym board upsert` gains `--record-file <file>`, parallel to `--body-file` in `internal/boardcli/bodyfile.go`.

- It is admitted only for an entry that already exists and carries a stored record.
- It is refused for a new entry, an entry with no stored record, a done entry, an entry a run holds, and an entry whose slug has a live batten run, held status or not.
  The run refusals' way forward names `lym loom decision replace <slug>` on the run, so a running raddle's record changes through D1.
- It is refused when the caller is a run's task session, by the board's existing classification of a run's agent; the operator's shell and orch strands may use it.
- The file must pass a record-only section check and the decision-id and acceptance check, both in `internal/discussionparser`, before anything is written.
  The record-only section check (new) checks the record's sections without a support log, since the board keeps none.
- The record and every other field of the payload land in one board write under the board lock, the record file written before `board.json` as `boardengine.Board.StoreDecisionRecord` does.

Bound: the flag rewrites a Sketch's design without a Discussion.
It is bounded to entries that already have a stored record and no run, to callers that are not a run's agent, and to a record that passes the format checks; git history of the Bolt keeps the old file.

Rationale: the operator keeps the Sketch's body and record current in one write, and raddle reads the record it has always read.

### D5 — A body hash on the stored record, and a promote warning when the body changed

Every write of an entry's stored record stamps the entry's `record_body_sha256` field (new) with the sha256 of the entry's body as that write leaves it.
The writers are `boardengine.Board.StoreDecisionRecord`, D4 and Raddle-File's filing write in `internal/boardengine/raddlefile.go`.
The field is part of the entry in `board.json`, so a board sync's replay carries it with the entry.
`lym board promote` of a Sketch adds a warning to its envelope when the sha256 of the entry's body, as loaded before the promote's write, differs from `record_body_sha256`.
The warning names `lym board upsert --record-file` as the way to bring the record up to the body.
It never refuses.
An entry with no `record_body_sha256`, stored before this task, gets no warning.

Bound: the warning fires on any body change after the record was stored, including one that leaves the design alone.

Rationale: the Sketch conversion, label edits and promote's own write leave the body alone, so only a body edit after the store warns.

### D6 — Specs and the parent directive say when to replace and when to add

`contracts/specs/loom-discussion-spec.md`'s "Parts no writer authors" keeps that a writer or fixer never edits or removes an added entry or appended bullet.
It adds that `lym loom decision replace` may, and that the dropped-entry check reads versions back to the newest replace.
`contracts/stencils/parent/parent-directive-parent.md` tells the parent three things:
- record a new call with `add`;
- change or withdraw an earlier decision on a run past its approval with `replace`, folding every added entry into the main decisions;
- once a motif run has made a Sketch whose record holds added entries, fold them with `lym board upsert --record-file`.
`docs/overview.md` names the replace verb and the `--record-file` flag where it names `add` and `store`.

This departs from issue #114's first expected point: no fixer folds.
An entry added in Discussion-Bouncer's settling window, after the last fixer round, would be removed without being folded, so the fold is the parent's explicit write.

Rationale: the verbs alone do not keep records current; the parent has to be told to fold.

## Constraints

- `PATTERN-run-records`: the project record's writer list grows by `replace`; see `## PATTERN changes`.
- `PATTERN-refusal-way-forward`: every new refusal of D1 and D4 names its way forward and lands with a test that reaches it.
- `PATTERN-fabric-boundary` and `PATTERN-gitrepo-client-boundary`: D1's commit and D2's trailer-bounded history are calls into `internal/fabricengine`.
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

## Acceptance criteria

- D1: a replace whose `--base-sha256` matches the committed record writes the file as the record, commits it with the trailer, and the record at HEAD equals the file.
- D1: a replace whose base differs is refused with a way forward, the refusal names no hash, and the record and HEAD are unchanged.
- D1: a replace on a run whose recipe has a Discussion approval row and that is not past it is refused, naming `lym loom decision add`.
- D1: a file that fails the section check or the decision-id and acceptance check, or that drops or changes a carry-over block, is refused and the record is unchanged.
- D1: a task session that is not the run's parent is refused, and the parent, the run's arbiter and the operator's unnamed shell are admitted, as for `add`.
- D1: a replace whose commit fails restores the prior record and refuses with a way forward that names `replace`.
- D1: an `add` started while a `replace` holds the decision lock runs after it, and its entry is in the record at HEAD.
- D1: each live strand of the run gets a notice naming the title.
- D2: after a replace that drops an `### Added after Discussion` entry, `lym loom validate-discussion` and the discussion gate report no `added-decision-dropped` finding.
- D2: when the replaced bytes landed in an earlier commit than the trailer commit, the check still reports no `added-decision-dropped` finding for the dropped entry.
- D2: an entry added after a replace and then dropped by a writer is still an `added-decision-dropped` finding.
- D3: `add` and `replace` on a raddle run of a promoted Sketch refresh the entry's stored record, and `lym loom decision store` is admitted on that run.
- D4: `lym board upsert --record-file` on an entry with a stored record and no run replaces the stored record in the same write as the payload's other fields.
- D4: `--record-file` is refused for a new entry, an entry with no stored record, a done entry, an entry a run holds, an entry with a live batten run, a run's task session and a file that fails the format checks, each with a way forward and nothing written.
- D5: promote of a Sketch whose body changed after its record was stored warns and still promotes.
- D5: promote of a Sketch made by a motif run with no body edit since, and of an entry with no `record_body_sha256`, gives no warning.
- Review: D6 — the discussion spec, the parent directive and the overview name `replace` and `--record-file`, when to use each rather than `add`, and the history reset.

## PATTERN changes

- `PATTERN-run-records` — changes: Writing a run's friction note, usage record or project record: only its one writer, `internal/friction`, shed's usage writer or `lym loom decision add` or `replace`. (test)

## Notes for the plan writer

- `add`'s preconditions, caller guards and notice code in `internal/loomcli/decision.go` are the base `replace` should share rather than copy; `add`'s amendment roll-back keys on added headings and does not fit a replace.
- Bounding `recordHistory` itself reaches every caller of the history seam.
- `Validate` writes nothing and reads the record from disk, so `replace` writes the file in place and restores the prior bytes on findings, as `AppendDecisions` does.
- The run lock lives in `internal/boardengine/runlock.go` and compares `board.json` fields only; D4's record refusals are a separate check beside it, while `boardengine.Board.StoreDecisionRecord` keeps its bypass.
