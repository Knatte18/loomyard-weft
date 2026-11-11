## Goal

lym board upsert --record-file replaces a Sketch's stored record, stamped with the body hash promote checks

Takes: the board's stored record files and Board.StoreDecisionRecord, the run lock in runlock.go, the Add weft probe in remoteleftover.go, and the strand-name caller classification in boardcli
Delivers: lym board upsert --record-file with its refusals and one-write landing, the record_body_sha256 stamp on every stored-record write, and promote's body-changed warning
Exports: a fabricengine function that reports whether a slug's weft branch is live, which Topology.Add also uses; a discussionparser record-only section check; the record_body_sha256 field on boardengine.Task

## Scope

The part changes these packages:

- internal/boardcli (about 250 changed lines)
- internal/boardengine (about 250 changed lines)
- internal/discussionparser (about 80 changed lines)
- internal/fabricengine (about 120 changed lines)

## Decisions

### D4 — `lym board upsert --record-file <file>` replaces an entry's stored record

`lym board upsert` gains `--record-file <file>`, parallel to `--body-file` in `internal/boardcli/bodyfile.go`.

- It is admitted only for an entry that already exists and carries a stored record.
- It is refused for a new entry, an entry with no stored record, a done entry, an entry a run holds, and an entry whose slug has a task worktree in the hub.
  The run refusal's way forward names `lym loom decision replace <slug>` on the run, so a running raddle's record changes through D1.
  The worktree refusal's way forward names `lym loom decision replace <slug>` in that worktree, or ending the run with `lym batten cancel <slug>` first.
  Record-Write never overwrites a record file the worktree holds, so a resumed run there would keep the stale record.
  A family project whose raddle run's filing is done, as the raddle store records, is exempt from the worktree refusal and from the live-weft refusal below:
  that run reads no record again, and D3 refuses `replace` on it.
- It reads no batten liveness.
  The run lock and the worktree check are its run guards.
- It is refused when the caller is a run's task session, by the board's existing classification of a run's agent; the operator's shell and orch strands may use it.
- The file must pass a record-only section check and the decision-id and acceptance check, both in `internal/discussionparser`, before anything is written.
  The record-only section check (new) checks the record's sections without a support log, since the board keeps none.
- The payload is validated before the record file is written.
  The record and every other field of the payload land in one board write under the board lock, the record file written before `board.json` as `boardengine.Board.StoreDecisionRecord` does.
  A failed save writes the entry's former record file back, as `boardengine.Board.FileRaddle` does.
- It is refused while the slug's weft branch is live by the test `fabricengine.Topology.Add` adopts by:
  a local branch, or one on origin with no `archive/<slug>/*` tag covering its tip.
  It applies that test as `Add` would at the moment of the upsert,
  so a pair whose detached teardown is still in flight is refused whenever a raddle start would adopt its weft.
  A raddle run that adopts that weft keeps the motif run's committed record, which Record-Write never overwrites, so the stored record would not reach it.
  The way forward is to wait for the pair's teardown to finish, or, when its remote step failed, to carry out the way forward that step's failure mail names, then re-run the upsert.
  The test is one general exported `fabricengine` function (new) that reports whether a slug's weft branch is live,
  built on the code `Add` already uses, the local branch check and the origin tip and archive-tag probe, so `Add` and the upsert cannot disagree.
  It probes origin only when `Add` would, under `Add`'s own skip-push and skip-git conditions;
  where `Add` makes no origin probe, the local branch check alone decides.
  When the origin probe fails, the upsert is refused, with the way forward to re-run it once origin answers.

Bound: the flag rewrites a Sketch's design without a Discussion.
It is bounded to entries that already have a stored record, no run, no task worktree in the hub and no live weft branch,
to callers that are not a run's agent, and to a record that passes the format checks; git history of the Bolt keeps the old file.
The run-agent refusal reads the strand name from the environment, so it guards against a mistaken call, not a forged or unset strand name.
It does not see a batten start after its check, or a batten run on another machine.

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
A record stored by a binary older than this task leaves `record_body_sha256` as it was,
so the warning can be wrong until a newer binary stores the record again.

Rationale: the Sketch conversion, label edits and promote's own write leave the body alone, so only a body edit after the store warns.

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

## Acceptance criteria

- D4: `lym board upsert --record-file` on an entry with a stored record and no run replaces the stored record in the same write as the payload's other fields.
- D4: `--record-file` is refused for a new entry, an entry with no stored record, a done entry, an entry a run holds, an entry whose slug has a task worktree in the hub, an entry whose slug's weft branch is live, an upsert whose origin probe fails, a run's task session and a file that fails the format checks, each with a way forward and nothing written.
- D4: `--record-file` on a family project whose raddle run has filed and keeps its worktree replaces the project's stored record.
- D4: a `--record-file` upsert whose payload the store refuses leaves the stored record unchanged.
- D5: promote of a Sketch whose body changed after its record was stored warns and still promotes.
- D5: promote of a Sketch made by a motif run with no body edit since, and of an entry with no `record_body_sha256`, gives no warning.
