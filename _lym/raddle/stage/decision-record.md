# Move a running task by a move note and a fresh run

## Goal

Give the orch one written procedure for moving a running task to another parent:
the run's live writer hands over what it learned for a move note, the orch holds the run,
checks that the new parent takes a member and cancels the run with `lym batten cancel <slug> --remove`,
then writes the note into what the entry's fresh run reads, sets the entry's new parent and has it released.
The fresh run reads the old run's commits and never re-uses them.

## Scope

In:

- A new section of the orch's beam role file, `contracts/orchrole/role-beams.md`, holding the procedure of D1 to D7.
- A pointer to it in the run loop of `contracts/orchrole/orch-role.md`, beside the cancel lines, and in that file's line naming what `role-beams.md` covers.
- One glossary line in `internal/heddleengine/glossary.md` (D8).

Out:

- Any new `lym batten` verb, flag or recipe row (D1).
- Any Go change to cancel, the board, fabric or the archive tags.
- Moving a beam project, a grouping project or a front task.

## Decisions

### D1 — Orch guidance alone, built on existing verbs

The move is a procedure in the orch's role text, run by the orch with verbs that exist today:
`lym creel post` to the writer and to another beam's orch, `lym tender pause` and `lym tender resume`, `lym batten cancel <slug> --remove` from the prime,
`lym board get`, `lym board upsert` and `lym board set-status` on the board, and `git -C` reads and fetches of the code and records repositories.
No batten verb, flag or recipe row is added, and it is never a recipe step.

Rationale: moving a running task is rare, and every step but the note is an existing verb.
The note's first three parts come from an agent session, which no verb can drive synchronously,
so a verb would only wrap a cancel and an upsert the orch already runs.
Bound: the procedure adds no path past a guard;
cancel keeps all its refusals, and the board accepts the edit only because cancel set the entry back to `draft`, which clears the run lock.

### D2 — Ask the writer, then hold the run

While no pause covers the slug, the orch asks the run's live writer, by `lym creel post --expect-reply <strand>`,
to answer with the move note's first three parts in its reply text, writing no file.
The live writer is the session of the row the run is in: the Discussion writer, the plan writer, or the Webster master session during Webster.
The note holds four parts:

- what is done, naming the cards or decisions it covers;
- the decisions taken, each with its reason;
- what is left;
- where the commits are: the task branch, its tip SHA and its merge base with the old parent branch.

The orch writes the move note in its own `.scratch/` from the writer's reply.
It then holds the run with `lym tender pause <slug>`,
and adds the fourth part itself, reading the task branch's tip SHA and its merge base with the old parent branch after the hold.
The note never sits in the task worktree, so no archive tag holds it, departing from the entry's "the archive tag keeps the hand-back";
the board copy of D4 is the durable one, and the orch's scratch copy covers the span until the board write.

Rationale: the entry is held by the run and takes no body edit,
and the Discussion writer and the Webster master may write only their own output files:
the Discussion writer its three files (`contracts/stencils/loom/loom-template-discussion.md`, section "What you may write"),
the Webster master its two contract files (`contracts/stencils/webster/webster-template-master.md`).
A session answers an ask outside its limits by naming the limit (`contracts/stencils/parent/parent-directive-parent.md`),
so an ask to write a file would come back refused, while a reply asks no writer for a file write.
`lym batten pause` would not hold the run: it sets only batten's own `pause_requested`, which batten reads at its own row boundary (`internal/battenshed/innerrun.go:400-401`, `:782-783`),
and the inner loom or Webster run reads only the pause record `lym tender pause` writes (`internal/webstercli/batchpause.go:33-54`, `internal/loomcli/pausehold.go`),
so under a batten pause it keeps making rows, batch commits and Parent-Align merge-ins.
The ask goes before the hold, since a pause that covers a strand holds every delivery into its pane (`internal/creelengine/doc.go:258`).
The orch reads the tip after the hold, so the note's tip SHA and merge base equal what the cancel archives;
a commit made between the reply and the hold is still in the diff D6 reads.
A slug-scope pause does not block the cancel, which refuses only a hub pause and lifts the slug's own pause entries when it completes (`internal/battencli/cancel.go:299-304`, `:572-576`).
Committing the note would need a new committing writer for no gain over the board copy.

### D3 — No live writer

When none of D2's writers is live, because the run is halted, sits in a review, gate or landing row, or its writer's strand is gone,
the orch holds the run with `lym tender pause <slug>` and writes the whole move note itself.
It reads the run's decision record, plan and status, and the task branch's log against the old parent, and fills the same four parts.
A live writer that refuses the ask, or replies without the three parts, is handled as no live writer.

Rationale: the procedure must not depend on a writer the orch cannot bring back without resuming the run it means to end.

### D4 — Order: note and hold, parent check, cancel, carrier and parent, release

1. The writer replies with the first three parts, and the orch writes the move note and then holds the run (D2),
   or the orch holds the run and writes the note (D3).
   Either way the orch records the tip SHA and merge base after the hold.
   The recorded tip is the worktree's tip of the task branch,
   or origin's tip when `git -C <prime> ls-remote origin` shows it strictly containing the worktree's, since the cancel then archives origin's tip alone (`archiveDiscardedTips` in `internal/fabricengine/taskbranchdiscard.go`).
2. The orch checks with `lym board get`, and for a beam also its ready record, that the new parent can take a member.
   A project qualifies when it is not done and no run holds it: its status is no run status (`<state> · <producer>`), or, for a beam, its producer is a hold-free row
   (Worktree-Create, Preflight, Record-Write, Beam-Up or Beam-Await; `internal/boardengine/runlock.go:44`, `internal/boardengine/project.go:99`).
   A beam on which `lym beam ready` has run never qualifies, since it never starts a member filed after it (`contracts/orchrole/role-beams.md:61`).
   That flag is no board field but a ready record, `_lym/landing/beam-ready.json` under the beam's anchor (`BeamReadyPath` in `internal/loomengine/config.go`, `internal/beamcli/cli.go:918-920`).
   `lym beam ready` commits it in the beam's run-records worktree with no push (`MarkReady` in `internal/beamcli/deps.go`, through `fabricengine.CommitAnchoredPaths`),
   so it sits on the beam's records branch `<beam branch>-weft` and never on the beam branch.
   lym reads the beam as flagged only when the record exists and its `slug` is the beam's slug (`ReadyFor` in `internal/pawlengine/ready.go`), and so does the orch.
   When the beam's worktree is on the orch's machine, the beam qualifies only when that worktree holds no such record under `_lym/landing/beam-ready.json`.
   Otherwise the orch runs `git -C <records repo> fetch origin`,
   then `git -C <records repo> show origin/<beam branch>-weft:<anchor>_lym/landing/beam-ready.json`,
   where `<anchor>` is the anchor's subpath with a trailing slash, empty for an anchor at the repository root.
   The records store's fetch refspec maps a one-repository hub's `refs/lym/heads/` to `origin/`, so the same read serves both layouts (`pattern/PATTERN-single-repo-hub.md`).
   A record found there that names the beam means the beam does not qualify.
   Any other outcome, an absent file or an `origin/<beam branch>-weft` that does not exist, proves nothing for a beam on another machine,
   since the record may be committed there and not pushed:
   the orch asks that beam's orch by `lym creel post --expect-reply` whether `lym beam ready` ran, and asks the operator when no reply comes within ten minutes;
   the beam qualifies only on a no.
   The root always qualifies.
   When the check fails, the move stops before the cancel: the orch lifts the pause with `lym tender resume <slug>` and tells the operator.
3. The orch runs `lym batten cancel <slug> --remove` from the prime, on the entry's owner machine.
   The two cases are told apart by the way forward the output names;
   whether the cancel ended anything does not show in its output, and every cancel step is a no-op once done (`cancelRun` in `internal/battencli/cancel.go`).
   - A partial finish is an ok envelope with `end_only: true` or a non-empty `way_forward`,
     or an error whose way forward is only to fix the cause it names, or to wait until origin answers, then run the cancel again.
     The orch takes the fix it names and runs the cancel again until it completes;
     when a run again returns the same error, the orch stops and tells the operator, its pause still in place.
   - A refusal is every other error: the hub-pause refusal, whose way forward needs a hub-wide resume,
     the owned-elsewhere refusal, "the work … reached its parent", "landed by another machine" and the nothing-left refusal.
     D7 handles every refusal.
   The orch never runs `lym tender resume` without a slug, since that lifts every pause entry of the hub, the operator's and its own slug hold alike (`internal/tendercli/cli.go:194`);
   on the hub-pause refusal it stops with its slug pause in place and tells the operator,
   and once the operator has lifted the hub pause it runs the cancel again.
   The code archive tag is `archive/<slug>/<first 12 hex of the tip>`, named from the tip recorded in step 1, never read from `archive_tags`.
   Before step 4, the orch confirms both sides of the teardown on origin:
   the code tag through `git -C <prime> ls-remote origin 'refs/tags/archive/<slug>/*'`,
   and no records branch of the slug through `git -C <records repo> ls-remote origin`.
   The records branch is the task branch's name with `-weft` appended (`RecordsBranchName` in `internal/fabricengine/branchname.go`),
   listed under `refs/heads/`, or under `refs/lym/heads/` in a one-repository hub (`pattern/PATTERN-single-repo-hub.md`).
   A discussed entry's `--record-file` upsert is refused while that branch reads live on origin (`internal/boardengine/recordupsert.go:38-46`),
   and a fresh run of any entry would adopt a live one, so the check holds for every entry.
   The teardown child pushes and deletes after the cancel returns, so the orch repeats the check until both hold or a remote-teardown failure notice for the slug arrives;
   on a notice, it takes the way forward the notice names, then checks again.
   When ten minutes pass with neither, the orch pushes the code tag by hand from the prime,
   `git -C <prime> push origin refs/tags/archive/<slug>/<12 hex>`, since the task worktree a notice names is removed, then checks again.
   When the records branch is still listed after that, the orch stops and tells the operator, naming the branch and the move note's path in its `.scratch/`;
   the entry stays a draft under its old parent, and the move goes on at step 4 once the branch is gone from origin.
4. The orch upserts the same entry (D5), writing a move note section into what the fresh run reads:
   - an entry with no stored decision record gets a `## Move note` section in its body;
   - an entry with a stored decision record (`discussed`) runs from that record, never its body,
     so the orch adds an H3 `### Move note` under the record's `## Notes for the plan writer`, adding that section as the record's last when it lacks it,
     and stores the record with `lym board upsert --record-file`.
     The note's four parts go in as bullets or H4 headings, never as H2.
   The section holds the note, the code archive tag and the fetch that brings it,
   `git fetch origin refs/tags/archive/<slug>/<12 hex>:refs/tags/archive/<slug>/<12 hex>`,
   the records archive tag's pattern `archive/<slug>/` in the records repository, and D6's read-only rule.
   In the same upsert, `project` is set to the new beam's slug, or emptied for the root.
   When the board refuses the `project` value, because the new parent moved past its hold-free rows after step 2,
   the orch runs the upsert again without the `project` change, so the entry stays a draft under its old parent with the move note written in,
   and tells the operator.
5. While the entry `stale-records-ref-blocks-rerun` has not landed, the releasing orch first runs a pruning fetch of the hub's records repository,
   `git -C <records repo> fetch --prune origin`, since the teardown child deletes the slug's records branch on origin apart from the code tag's push:
   a stale tracking ref of the deleted records branch makes dobby refuse the re-run with `the code tip ... is not on origin yet`, and the prune clears it (that entry's body).
   The entry is then released with `lym board set-status` and a null status by the orch whose worktree is the new parent.
   When the new parent is another orch's worktree, the moving orch leaves the entry `draft` and tells the operator which orch releases it, and that the prune comes first.

Rationale: `--remove` is the only cancel that makes archive tags and lets the slug run again (Constraints).
The parent check comes before the cancel, since the board refuses a `project` that is done or held by a run, and a cancelled task under its old parent is no move.
Releasing stays with the orch that owns the new parent, as the orch role already says for every entry.

### D5 — The same entry carries on, not a new task

The moved work stays on the same entry and slug; the orch files no new task for it.

Rationale: after `--remove` the entry is a `draft` with no pair, and nothing but its `project` field records a parent,
so changing `project` is the whole move on the board side.
The entry keeps its handle, history, labels and run settings.

### D6 — The fresh run reads the old commits, never re-uses them

The fresh run forks from its new parent's head like any run.
It reads the old work through the code archive tag:
it first fetches the tag, `git fetch origin refs/tags/archive/<slug>/<12 hex>:refs/tags/archive/<slug>/<12 hex>`,
since a plain fetch never follows an archive tag whose branch was deleted and the code store keeps git's default refspecs (`pattern/PATTERN-single-repo-hub.md`),
then reads the diff from the note's merge base to the tag.
No agent cherry-picks, merges or checks out the archive tag into the new task branch;
work the fresh run keeps is written again through its own plan and gates.
A move before any code commit leaves an empty diff, and the note alone carries the old run's work.

Rationale: a cherry-pick or merge of the old tip brings back the study's ancestry question (`docs/research/loom/reparent-running-task-2026-11-12.md`, section 3),
since the old tip can carry unlanded commits of the old beam into a parent that never took them.
Reading the diff keeps what was learned and puts every kept line through the new run's review.

### D7 — When a move is refused

The procedure moves a task only through a cancel that completes.
When cancel refuses, as D4 step 3 defines a refusal, the move stops there:
the orch lifts its pause with `lym tender resume <slug>`, and takes the way forward cancel's message names;
for work that already reached its parent, that is letting the landing finish, after which there is nothing to move.
On the hub-pause refusal the orch keeps its slug pause and tells the operator instead, as D4 step 3 says.
On the owned-elsewhere refusal `lym tender resume <slug>` is refused too, since it refuses a run another machine owns,
so the orch tells the operator, who chooses between the adopt and the cancel on the owner that the refusal names.
A beam project, a grouping project and a front task are never moved this way.

Rationale: cancel's refusals already guard the states where a move would lose or duplicate landed work.

### D8 — The term `move note`

The note of D2 is called the move note, never a hand-back, since the shed driver already calls every non-`done` stop a hand-back.
The glossary gains the line:
`- **move note**: what a cancelled run learned, written for the fresh run that replaces it. Never: hand-back, handoff.`

Rationale: the entry's word "hand-back" collides with the driver's stop report and the tackler's `hand-back` verdict.

## Constraints

- `lym batten cancel` sets a run-form status back to `draft`, which frees the board's run lock (`internal/boardengine/runlock.go`).
- With `--remove`, cancel creates the archive tags locally, under `archive/<slug>/<12 hex of tip>` in each branch's own repository,
  and a detached teardown child pushes them and deletes the branches on origin after the command returns;
  a failed push mails the orch a remote-teardown notice naming the manual tag push
  (`internal/fabricengine/taskbranchdiscard.go:24-41`, `:79-97`, `internal/fabricengine/remoteteardown.go:184-207`, `:274-287`, `:319-333`).
  So `archive_tags` names at most the local tip's tag, created but not yet pushed.
- Cancel without `--remove` keeps the pair and makes no archive tags, and the slug refuses to run again until `--remove` runs.
- A task that never ran, or whose pair was removed, records its parent only in the board entry's `project`;
  `_lym/fabric/origin.json` exists only while a pair exists.
- A task's owner machine is read through its parent (`internal/boardengine/owners.go`), so a moved task's fresh run starts on its new parent's owner;
  it fetches the code archive tag from origin, where D4 step 3 confirmed it, and needs nothing else local to the old machine, since the note is on the board.
- Cancel refuses on a machine other than the entry's owner (`PATTERN-owner-machine`).
- An entry with a stored decision record runs from that record and never reads a body edit (`internal/boardengine/decision.go`).
- `PATTERN-stencil-ownership` and `PATTERN-heddle-instructions`: the procedure lives in the orch role text, and the glossary line in heddle's shipped glossary.
- No PATTERN entry changes: the move note is no run record a sole writer owns, since nothing commits it.

## Auto-mode assumptions

- Guidance alone, not a verb (D1), picked without the operator.
- The term `move note`, its first three parts handed over in the writer's reply and the note written in the orch's `.scratch/` (D2, D8), picked without the operator.
- The note is never committed nor written in the task worktree, departing from the entry's archive-tag wording (D2), picked without the operator.
- The same entry carries on rather than a new task (D5), picked without the operator.
- Read-only use of the old commits (D6), picked without the operator.

## Open risks

- The re-run of D5 depends on D4 step 5's pruning fetch until `stale-records-ref-blocks-rerun` lands.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/discussion/round-3-review.md; fixer report: _lym/reviews/discussion/round-3-fixer-report.md.
  - no-check-new-parent-takes-member (class design, severity MEDIUM, open in rounds 1, 2, 3)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- Review: `contracts/orchrole/role-beams.md` holds a section on moving a running task to another parent that states guidance alone with existing verbs and no recipe step (D1).
- Review: that section has the orch ask the live writer by creel, while no pause covers the slug, for the first three parts in its reply;
  write the note in the orch's `.scratch/`; hold with `lym tender pause <slug>`; and add the tip SHA and merge base read after the hold (D2).
- Review: that section has the orch hold the run and write the move note itself when none of the writers is live,
  or when a live writer refuses the ask or replies without the three parts (D3).
- Review: that section gives the order note and hold;
  the new-parent check, with `lym board get` and, for a beam, the read of `_lym/landing/beam-ready.json` on the beam's records branch,
  and for a beam on another machine whose record origin does not show, the ask to that beam's orch or the operator, the beam qualifying only on a no,
  that stops the move before the cancel;
  `lym batten cancel <slug> --remove` from the prime, with a partial finish and a refusal told apart by the way forward each names,
  and no `lym tender resume` without a slug, the hub-pause refusal stopping with the slug pause in place until the operator lifts the hub pause;
  the code archive tag named from the recorded tip, and before the board write both the tag confirmed on origin by `ls-remote` and no `<task branch>-weft` records branch listed on origin,
  with the ten-minute bound, the manual tag push from the prime and the stop that names the note's scratch path;
  the move note section in the body or, for an entry with a stored record, as an H3 under `## Notes for the plan writer` through `--record-file`,
  naming the tag fetch the fresh run makes;
  the `project` change, and the upsert without it that keeps the note when the board refuses the `project` value;
  the pruning fetch of the records repository while `stale-records-ref-blocks-rerun` has not landed; and release by the new parent's orch (D4).
- Review: that section keeps the same entry and files no new task (D5).
- Review: that section forbids cherry-picking, merging or checking out the archive tag,
  and has the fresh run fetch the tag with `git fetch origin refs/tags/archive/<slug>/<12 hex>:refs/tags/archive/<slug>/<12 hex>` and read the diff from the merge base (D6).
- Review: that section stops the move when cancel refuses, lifting the pause with `lym tender resume <slug>` except on the hub-pause and owned-elsewhere refusals,
  and excludes beam projects, grouping projects and front tasks (D7).
- Review: `internal/heddleengine/glossary.md` holds the `move note` line of D8.
- Review: `contracts/orchrole/orch-role.md`'s run loop points to the new section beside the cancel lines, and its line on `role-beams.md` names moving a running task (D1).
