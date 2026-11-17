# Decision record — review rounds converge

## Goal

Burler's review rounds stop circling in three ways the board entry names.
A fixer's rebuttal reaches the next reviewer beside its finding, and the reviewer withdraws the finding or answers it.
A round after the first checks the earlier round's findings and raises a new finding only on what the fixes changed,
and only at BLOCKING severity.
The Bouncer stops a segment with a named `stalemate` cause, by a Go check,
once a set number of rounds in a row left the review target unchanged.
The entry names the warp tree; this record reads it as the review target,
so Discussion-Review and Plan-Review, whose fixers edit `_lym` documents, are covered too.

## Scope

In:

- The burler review and fix halves of every review segment the Bouncer drives: Discussion-Review, Plan-Review and Code-Review.
- The fixer-report format, the review-file format, the reviewer's and fixer's stencils, and their Go checks.
- The Bouncer's stalemate check, its escalation cause, the loom config key that sets its count,
  and the `lym loom circling` verbs that settle it.
- Every reviewer engine: Claude and Codex render the same rules.

Out:

- Finalize's fix reviews, which `internal/landingshed` runs outside burler.
- The `circling` cause and its LLM judge: they stay as they are, beside the new Go check.
- The bounce budget, the hard cap and amendment rounds' own scoping, apart from D4's full-round rule.

## Decisions

Round N is the round under work; round N-1 is the round before it.

### D1 — The fixer-report carries a parsed disposition per finding, rebuttals included

The fixer-report gains `---`-delimited YAML frontmatter over its existing prose.
Its one key, `dispositions`, lists one entry per finding of the round's review the fixer owns: every finding but a `decision`-class one.
Each entry has `id` (the review's finding id) and `disposition`, one of `fixed`, `fixed-differently`, `disputed` or `deferred`.
A `decision`-class finding may carry a `ruling` entry when a parent ruling reached the fixer; it is allowed, never required,
since no Go record says which findings a ruling reached.

- A `disputed` entry also carries `ground` (`premise false` or `every fix worse`), `rebuttal` (why the code stands)
  and `evidence`, a non-empty list of `<path>:<line>` locations the rebuttal rests on.
  A dispute resting on verify output cites the test file and line that output comes from;
  that narrowing of `contracts/stencils/burler/burler-step-fix.md`'s evidence kinds is deliberate, since the entry asks for file and line.
- A `fixed-differently`, `deferred` or `ruling` entry carries `reason`.
- The dispute grounds, the disposition order and the deferral rule of the fix step stay as they are;
  the frontmatter states their outcome in a form Go reads, and the prose sections stay for the reader.
- The fixer-reports Go writes itself when no fixer is spawned carry the frontmatter too, with the dispositions their case implies:
  empty when every finding was routed to the parent.

`internal/burlerengine` is the sole parser of the frontmatter, beside its review parser.
A fix check joins the fix half's gate as the review check joins the review half's:
it refuses a report whose frontmatter is missing or malformed, or whose dispositions miss, repeat or invent a finding id.

Rationale: the rebuttal must reach the next round by Go, not by the next reviewer finding and reading a prose file,
and only a parsed format lets Go place each rebuttal beside its finding.

### D2 — The next reviewer gets the earlier round's findings, each with its disposition and rebuttal beside it

When round N-1 is complete in the run directory, its review and its fixer-report both present,
Go renders a prior-findings block into round N's `prior_rounds` value, replacing today's prior-rounds text.
The block lists every finding of round N-1, `decision`-class ones included,
with its severity, class, location, summary and fix, and under it the fixer's disposition:
for `disputed`, the ground, the rebuttal and the evidence; for the others, the reason;
for a `decision`-class finding with no `ruling` entry, that it is with the parent.
It names round N-1's review and fixer-report paths, and D4's changes file when one exists.
A round with no complete round N-1 keeps today's prior-rounds text.

The rules the reviewer follows on the block live in `contracts/stencils/burler/burler-step-review.md` and the review-file format,
outside any engine block, so the Claude and Codex reviewers read the same rules.

Rationale: the board entry asks for the rebuttal beside its finding, for every engine;
Go-rendered text from parsed files is the one channel every engine's reviewer reads the same way.

### D3 — The review file records each earlier finding's status and links each carried finding

When D2's block is rendered, round N's review frontmatter gains a `prior` list, one entry per finding the block lists:
`id` (round N-1's id) and `status`, one of `resolved`, `open`, `outdated` or `withdrawn`.

- `withdrawn` is legal only for a finding round N-1's fixer disputed: the reviewer accepts the rebuttal.
- `open` on a disputed finding needs `answer`: why the rebuttal does not hold, citing what the reviewer read.
- `outdated` means the code the finding named is gone or rewritten so the finding no longer applies.
- Each `open` entry is carried as exactly one finding in round N's `findings`, which names it in a new `carries` key
  (`carries: F3`), so the fixer's flow and every reader of `findings` stay unchanged.
  A carried finding keeps or lowers its severity, never raises it.
  An open `decision`-class finding is carried the same way, so a question still with the parent stays in the review the Bouncer reads.

The review check in `internal/burlerengine/verdict.go` refuses a review whose `prior` list misses, repeats or invents an id,
holds `withdrawn` on an undisputed finding, lacks `answer` on an open disputed one,
or whose `carries` links do not match its `open` entries one to one.
A review with no D2 block carries neither key, and the check refuses either there.
`lym burler validate-review` finds round N-1 from the review file's run directory and round number and runs the same checks;
a file outside a run directory gets the shape checks alone.

Rationale: the statuses make the re-review's outcome explicit and checkable, and `carries` keeps a still-open finding's identity across rounds
without changing what the fixer reads.

### D4 — A round after the first raises a new finding only on what the fixes changed, and only at BLOCKING

A round is incremental when D2's block is rendered and the round does not review an amendment.
Any other round is full: a segment's first round, the first round of a new generation (the Bouncer archives the run directory then,
so no complete round N-1 remains) and an amendment's first round.

At the start of an incremental round, the review producer writes a changes file into the round's run directory:
the diff of the review target from round N-1's snapshot (D5) to now.
Its paths are worktree-relative.

In an incremental round, a finding with no `carries` key is new, and the review check refuses it unless it is `BLOCKING`.
When the changes file exists, the check also refuses a new finding whose location's path is not a file the changes file touches.
The location's path is its text up to the first `:`, `#` or space, made relative to the worktree root;
the format tells the reviewer to open every location with the file's path.
When the snapshot's cache is gone and no changes file can be built, the BLOCKING rule holds alone.
A finding with `origin: parent` is exempt from both rules: the parent's ask sets its scope.
The stencil tells the reviewer to raise a new finding only on a changed hunk; the check holds the file bound.
Cluster and assisted rounds keep their fan; forks and assistants get the same rule, and the review check judges the consolidated review.

Bound: the checks join the review gate, which passes on its attempt cap as it does today;
a review that passes on the cap keeps its out-of-bound findings, and the fixer works them.
Short of the cap, an incremental round raises no non-BLOCKING finding and no new finding on an unchanged file,
so a defect round 1 missed in code no fix touched stays unraised by later rounds of that segment.
In source scope an untracked, non-ignored file the fixer leaves counts as touched.

Rationale: a later round that reviews the whole target again finds new nits on code it passed, which is the circling the task removes.

### D5 — Go records a snapshot of the review target once per round

The review producer writes a small snapshot record into the round's run directory before the reviewer first starts, through `internal/state`.
A review half that runs again in the same round keeps the record it finds, so a dead fixer's partial edits never become the round's start.
The record holds the fix scope and a target id:

- `source` scope (Code-Review): the id of the warp worktree's working tree, uncommitted edits included,
  taken through a new general fabric verb that hashes what `git add -A` would stage under the warp's excludes into a tree object,
  without touching the index or refs.
  A second general fabric verb diffs two such tree ids, for D4's changes file.
- `overlay` scope (Discussion-Review, Plan-Review): a sha256 over the target files' sorted worktree-relative paths and bytes.
  A copy of those files goes under `.lym`, at a path `lymdirs` names, so D4 can diff them; Go builds the diff itself.

The record is tracked with the run; the tree object and the `.lym` copy are caches.
A missing cache leaves D4 without a changes file and never fails the round.

Rationale: the ring runs the review half, then the fix half, then the Bouncer, and the reviewer never writes the target,
so the target at the review's start is the target the fixer starts from.
One per-round record serves D4's diff and D6's count;
the working-tree id, not HEAD, keeps an uncommitted fix from reading as no change.

### D6 — The Bouncer escalates a stalemate before the budget runs out

`shedadapters.EscalationCause` gains a fifth value, `stalemate`.
Round N is unchanged when the target id the Bouncer computes after round N's fix half, through D5's seam,
equals the id round N's snapshot record holds, and a fixer ran in round N.
A round with no snapshot record, or whose fixer-report Go wrote because no fixer was spawned, counts as changed.
The Bouncer counts the unchanged rounds in a row ending at round N,
starting after the latest round that escalated with cause `stalemate`; an escalation of another cause restarts nothing.

The check runs in the Bouncer's unconverged path: after the judge returns a verdict that does not converge, Go guards included,
and before the circling, soft-cap and hard-cap rules.
The judge still runs and writes its ledger every round, so the ledger chain and convergence stay the judge's.
The decision escalation keeps its place ahead of the judge.
Once the count reaches the loom config key `review_stalemate_rounds`, the Bouncer escalates with cause `stalemate`,
writing the escalation record and parent notice as it does for `circling`, and returns Awaiting.
That escalation is how the segment stops: it runs no further round without the parent.

`lym loom circling accept` ends the segment on the current round, and `continue` runs another round and restarts the count;
both settle a `stalemate` escalation as they settle the other four causes, and `continue` keeps its hard-cap refusal.
The parent notice names both verbs as the way forward.

`review_stalemate_rounds` defaults to 2; 0 turns the check off.

Bound: a stalemate escalation is Awaiting like `circling`, never counts against the bounce budget,
and ends no segment without the parent's `accept`.

Rationale: a round whose fixer changed nothing is a fact Go can read, so it needs no LLM judgment of its own;
two in a row means a rebuttal round (D1 to D3) settled nothing.

## Constraints

- `PATTERN-fabric-boundary`: D5's two verbs are general fabric verbs, a working-tree id and a diff of two ids, shaped for no consumer;
  the Bouncer and the review producer reach them through closures on their configs, as the Bouncer's other seams are wired.
- `PATTERN-sole-parsers`: `internal/burlerengine` is the sole parser of the review file and of the fixer-report frontmatter,
  and the Bouncer and every other consumer read its model.
  Both formats join the entry's background list in a Webster card; the entry's own line is unchanged.
- `PATTERN-lym-files`: D5's overlay copy is a never-tracked cache under `.lym`, named only by `lymdirs`.
- `PATTERN-tolerant-records`: D5's snapshot record is Go-written JSON through `internal/state`;
  the review and fixer-report frontmatter are agent-written and stay strict.
- `PATTERN-config-layers`: `review_stalemate_rounds` is classified `CategoryNone` in loom's `ConfigSpec`:
  its value only moves an escalation to the parent earlier or later and lets no gate pass.
- `PATTERN-stencil-ownership`: the new rules live in the burler stencils, read through `stencilstore`, outside engine blocks.
- `PATTERN-refusal-way-forward`: the stalemate parent notice and every new refusal of the review and fix checks name the way forward,
  and a test reaches each.

## Auto-mode assumptions

- D1: frontmatter on the fixer-report, not a new file; verify-output evidence cites a test file and line.
- D3: an open earlier finding is carried with `carries`, decision-class ones included.
- D4: Go checks the file bound; the hunk bound is the stencil's.
- D4: `origin: parent` findings are exempt from the incremental bound.
- D5: the source-scope snapshot is the working tree's id, not HEAD.
- D6: the check runs after the judge, so convergence stays the judge's.
- D6: one loom key `review_stalemate_rounds`, default 2; the existing `lym loom circling` verbs settle it.

## Open risks

- An in-flight run that crosses the binary change has rounds with no snapshot record or fixer-report frontmatter:
  its next round is full and counts as changed, so it reviews and bounces as today until it holds a complete new-format round.

## Acceptance criteria

- D1: a fixer-report whose dispositions miss a finding the fixer owns, repeat one, or name an id the review lacks is refused by the fix check;
  one with a `disputed` entry lacking `rebuttal` or a `<path>:<line>` evidence item is refused; a complete report passes.
- D1: a `decision`-class finding needs no disposition and may carry a `ruling` entry; the Go-written fixer-reports parse.
- D2: round N's composed review prompt holds every finding of round N-1, each with its disposition and, for a disputed one,
  its rebuttal and evidence, for both the Claude and the Codex reviewer engine;
  a round with no complete round N-1 holds today's prior-rounds text.
- D3: the review check refuses a review whose `prior` list misses or invents an id, holds `withdrawn` on an undisputed finding,
  lacks `answer` on an open disputed finding, or whose `carries` links do not match its `open` entries;
  it refuses `prior` or `carries` in a review with no D2 block.
- D3: a carried finding whose severity is above its earlier finding's is refused.
- D3: `lym burler validate-review` on an incremental review in its run directory runs the round checks; on a file outside one, shape checks alone.
- D4: in an incremental round, the review check refuses a new non-BLOCKING finding and a new BLOCKING finding on a file the changes file does not touch,
  and accepts a new BLOCKING finding on a touched file, located by worktree-relative path with a `:line` or `#` suffix, and an `origin: parent` finding anywhere.
- D4: with no changes file, a new BLOCKING finding on any file passes and a new non-BLOCKING one is refused.
- D4: a segment's first round, a new generation's first round and an amendment's first round are full rounds.
- D5: a round's snapshot record holds a target id that changes when a fixer edits a target file, committed or not,
  and stays the same when nothing changed; a review re-run in the same round keeps the first record.
- D5: the changes file of an incremental round lists exactly the target files changed since round N-1's snapshot, for source and overlay scope.
- D6: with `review_stalemate_rounds` at 2 and a judge verdict that does not converge, a second unchanged round in a row escalates with cause `stalemate`;
  one unchanged round does not; a changed round between two unchanged rounds resets the count; 0 never escalates.
- D6: a converged verdict on an unchanged round converges; a round with no snapshot record, or whose fixer was not spawned, counts as changed.
- D6: `lym loom circling continue` after a stalemate runs another round and the count restarts; `accept` ends the segment.
- Review: `contracts/stencils/burler/burler-step-fix.md` tells the fixer to write the D1 frontmatter,
  and the review-file format states D3's `prior` and `carries` keys and D4's bound.
- Review: the stalemate parent notice names `lym loom circling accept` and `continue`.
