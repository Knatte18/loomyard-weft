# Decision record — arbiter-fixes

## Goal

Fix three rough edges around the arbiter and the alignment agent.
A re-launch whose arbiter verdict is already in effect is admitted instead of refused as moved, and the second actor that moved the run while its arbiter was live no longer races it.
The Webster deviation list a reader sees comes from the glyph scope guard alone, never from a fork's guess.
The alignment agent settles cross-run landing order itself and escalates to the orch only on a design conflict.

## Scope

In: the board entry's three sections, `arbiter-relaunch-moved` with its "Also seen" item, `webster-deviations-list` and `alignment-landing-order`, plus the docs, refusal-spec rows and tests each change carries.

Out:

- The wider narrowing of arbiter triggers (hard cap, circling, every blocked stop that names a goto): `arbiter-when` owns it.
- Any change to batten's verify-retry mechanics, the Go driver's rule table or the LLM driver's park and notify rules.
- `ScopeGuard`'s informational posture and record-batch's other checks; D4 changes only what the guard reports and where its result goes.
- The alignment verdict file's keys and `ReadVerdict`.

## Decisions

### D1 — A re-launch whose verdict is already in effect is admitted

Decision: the consume of a `resume` attempt admits a run whose stop key moved since the spawn when the move is the verdict itself.
That holds when the verdict names a goto target, the run's current producer is that target, its state is `paused`, every history entry after the attempt's recorded history length is a `goto` entry and the last of them names the target.
Such a re-launch marks the attempt applied and performs no second goto, so the history gains no duplicate entry; the loop then steps from the row the run already stands on, under the hub pause like any step.
On this admit only, the pause check refuses only a pause condition present now that the attempt did not record at spawn; a condition the goto cleared does not refuse.
An unmoved run keeps today's equality check, so a pause condition cleared by hand since the spawn still refuses its re-launch.
The arbiter's and the tackler's consume share one admit check, so the two cannot drift apart.
`shedengine.Relaunch` learns from its admit hook that the target is already in effect and skips its goto; the signal is generic and names no loom type.
Bound: besides the verdict's own move, this lets through a parent's goto to the same row, which the re-launch then steps without waiting for the parent's resume, as the Go driver's recheck already does for an orch goto.
A step run after the goto, a different row, a resume with no goto and an already-applied attempt are refused as today, under the same status lock.

Rationale: the refusal parked runs for the parent although the run stood where the verdict wanted it, and the parent's only action was the same re-launch by hand.

### D2 — The mover is batten's verify retry, and the arbiter refuses its Merge-In stop too

Decision: the second actor is batten's first-verify-failure retry (`internal/battenshed`'s publish retry), which moves a child blocked at Merge-In or Publish to its failure record's target and resumes it, independent of the child's driver.
The arbiter verb already refuses a `blocked` Publish halt whose verify-failure record names a target, as belonging to batten's retry and then to the parent.
That refusal widens to Merge-In: a `blocked` stop at Merge-In or Publish whose verify-failure record names a target and belongs to the halt being judged is not eligible.
The record belongs to the halt when its row is the run's current producer, its head is the pair's HEAD and it was written during the halting attempt, after the history entry before the halt and no later than the halt, the same match battenshed's retry makes before acting.
The Publish check takes the same match, since the record outlives its halt until a later passing verify removes it.
The existing refusal-spec row is edited to name both rows; no second row is added.
The LLM driver acts on it through its existing arbiter-refusal path and parks; its stencil gains no new rule, and batten's retry, or the parent when no batten retries, takes the run on.
Bound: the refusal removes the arbiter only from a Merge-In verify halt whose own failure record names a target, the case Publish already has; every other blocked stop at Merge-In and at the other rows keeps its eligibility, a stale record from an earlier verify included.
The Publish refusal narrows: a blocked Publish stop of another cause, such as a rejected push or a tree dirtied after the verify, with a stale verify-failure record naming a target becomes eligible to the arbiter where it is refused today.
With no batten, or with batten's one retry spent or capped, a Merge-In verify halt whose record names a target is no longer settled by the arbiter and parks for the parent, as the Publish halt already does.

Rationale: at such a stop the arbiter's verdict can only restate the goto the failure record names, and the Go driver's watched-verify hand-back plus batten's retry already cover it, so the spawn only races batten.
The Publish refusal shows the rule already chosen for the same stop shape.

### D3 — Stencils name the board payload

Decision: the arbiter stencil's read list gives `lyx board get '{"slug":"{{.slug}}"}'`, the JSON payload form the verb takes.
The alignment stencil's two `lyx board get` lines name the same payload form.

Rationale: the bare-slug wording led the arbiter to a JSON parse error, and the alignment stencil, edited by D5, carries the same unspecified form.

### D4 — The scope guard alone produces the deviation list

Decision: the implementer fork's report drops `deviations`; it reports `status` and `head_sha` only.
`ParseReport` keeps accepting a `deviations` key from an older fork and ignores it.
Record-batch derives the batch's deviation list from the scope guard over the batch delta and stores it in the batch digest's existing `deviations` field, as the deduplicated, sorted list of worktree-relative files.
The guard's finding carries its file as a structured field, never parsed out of the detail text.
The guard also reports, at file level, a changed file the delta gives no symbols for, unless a card target's unit covers it, the unit taken through `Glyph.UnitPath`, and for an external test package through planparser's one external-test unit mapping.
A file-unit target covers the one file `UnitPath` gives, compared by equality.
A package-unit target, every Go symbol target among them, covers every file directly in the package directory `UnitPath` gives, never a file in a subdirectory.
A symbol target whose unit is an external test package, which quarry names `<dir>_test` with no directory of its own, covers every file directly in the package directory that planparser's external-test unit mapping gives, by the same rule.
The scope findings no longer also ride record-batch's warnings, so a reader meets them once, in the digest.
The digest keeps its file list, so the symbol names the guard's findings carry are shown nowhere once they leave the warnings.
When the delta could not be computed, the digest carries no list and the existing guard-could-not-run warning stays.
The Master stencil, the fork prompt's predecessor digest line and the Master's summary narrative at its summary path name the digest's list as the batch's deviations, with the guard as its source.
The list stays informational: it never makes a batch `stuck` or `FAILED`.
Bound: the list lets through any change the delta gives no symbols for (imports, file header, comments) in a file a card's file target names, and in any file directly in a package directory a card's symbol or package target names, an external test package's symbol target naming its package directory through planparser's external-test unit mapping, non-Go files in that directory included.
It reports at file level every other changed file the delta gives no symbols for, unsupported and extraction-error dispositions included.
The reader surface drops to file granularity: the list names which files changed outside the targets, never which symbols.

Rationale: a fork cannot map a glyph to its file without the comparison the guard makes, so its list mixed real deviations with the batch's own files; the guard runs over every batch delta, and the file-level comparison keeps the non-code files the fork's list used to carry.

### D5 — The alignment agent settles landing order itself

Decision: an overlap with a peer run or a landed parent commit no longer escalates on its own.
The agent applies the landing-order rule: the run furthest along by its recipe row lands first, and the other takes its changes at Merge-In.
The agent reads the row order from `lyx loom peers`, which serves it from the loom recipe the running binary embeds, so a hub with no loomyard checkout has the same source and the stencil restates no rows, per `PATTERN-producer-pointer-rule`.
Each peer line gains `row`, the 1-based position of its run status's producer in that recipe, 0 when the producer is not a loom row or the status is unreadable; from a task pair the list also gains `own_row`, the caller's own position by the same rule.
Every peer is read against the loom recipe, darn runs included: a darn run at a row name it shares with loom, such as Merge-In, Publish or PR-Gate, takes that row's loom position, and one at a darn-only row reads 0, an unknown order.
The agent compares each peer's `row` against `own_row`; a peer whose `row` is 0 is an unknown order.
It approves (or rejects on a scope deviation) and records in its report each overlapping peer, what overlaps and the landing order the rule gives; two peers on the same row, or a peer whose status is unreadable or not a loom row, are reported as a tie or an unknown order, and the verdict is unchanged.
It escalates on an overlap only when the designs conflict: both runs change the same behaviour in different ways, a recorded decision in one contradicts or duplicates one in the other, or both change the same PATTERN entry in different ways.
A landed parent commit holds no decision record, so the agent compares the commit's subject and diff against this run's decision record and diff, and escalates when the commit changes the same behaviour or the same PATTERN entry in a different way than this run's decisions do, or already makes one of this run's decisions, matching the peer rule's duplicate case.
A text-only overlap, the same file or package changed at different places, is never a reason to escalate.
An open question that needs a decision the board and the records do not give still escalates, as today.
The rule applies at both `parent-review` and `pr-gate`.
Bound: the agent gains no new action; it only stops escalating text-only overlaps, and the parent lock and Merge-In already make whichever run lands second take the first's changes.
A text-only overlap that interacts in behaviour, such as one run changing a signature while the other adds a caller elsewhere in the package, is caught only by the verify at the second run's Merge-In, and two runs changing the same PATTERN entry the same way no longer escalate.
Both hold at `parent-review` and at `pr-gate`, where an approve lands the run with no orch turn.
The `row` and `own_row` fields are read-only: they reveal only the recipe positions of the peers' and the caller's producers, which their board statuses already name.

Rationale: the orch settled every such escalation of 2026-10-12 by this one rule with no design call, and the operator wants the orch woken only for stops that need a parent choice.

## Constraints

- This task builds on `go-driver-assistant`, landed: the tackler consume and the Go driver's verify facts are its work.
- `PATTERN-refusal-way-forward`: D2 edits the existing Publish refusal row in `contracts/specs/refusal-spec.md` and its test; D1 narrows the trigger of the "run moved or pause conditions changed" consume row and the tackler's matching row.
- `PATTERN-shed-producer-seam`: D1's skip signal in `shedengine` stays generic; the verdict-in-effect logic stays in `loomcli`.
- `PATTERN-glyph-conversion-chokepoint`: D4's file-level comparison takes a target's file through `glyph`'s conversions and planparser's one external-test unit mapping only.
- `PATTERN-stencil-ownership`, `PATTERN-producer-pointer-rule`: the stencil edits stay stencil-store files read at call time; the Master stencil points at the digest, never restates the guard's comparison.
- `PATTERN-tolerant-records`: the digest stays a Go-written JSON record rewritten through `internal/state`; a missing `deviations` field reads as an empty list.
- `PATTERN-parent-lock`: D5 relies on the parent lock and Merge-In for landing order and changes neither.
- CLI output change: D5 adds `row` to each `lyx loom peers` line and `own_row` to the list, documented in `internal/loomcli/doc.go` (# Peers); no verb is added, so `docs/overview.md` is unchanged.
- New top-level tests stay within each package's row in `cmd/lyx/testdata/test-budget.yaml`, or the card lists that file under `Edit:`.
- Docs land in the same commit: `internal/loomcli/doc.go` (# Arbiter eligibility and consume, # Tackler consume), `internal/websterengine/doc.go` (fork-return contract), `internal/planglyph`'s scope doc, `internal/alignment/doc.go`.

## Auto-mode assumptions

- The incidents' arbiter attempt records are gone, so D2's mover rests on the code path and the incidents' shape.
- The tackler's consume takes D1 too, since it is the same re-launch shape over the same status lock.

## Open risks

- `arbiter-when` later refuses every blocked stop whose envelope names a goto; it extends D2's widened check and its spec row rather than adding another.
- If the mover was a different actor in either incident, D1 still admits the re-launch, but D2 would not have closed that incident's race.
- D4 removes the fork-reported list that `glyph-use-audit` planned to compare against the guard; runs after this lands give that audit question nothing to compare.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - d4-external-test-unit (class design, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- A `resume` attempt with a goto, over a run moved by a goto to that target after the spawn, is consumed: the attempt is marked applied, the history gains no second goto entry and the loop steps from the target.
- The same case where the attempt recorded a pause condition at spawn that the goto cleared is consumed too; a pause condition present now and absent at spawn is refused as paused.
- An unmoved run whose pause condition was cleared by hand since the spawn is refused as paused, as today: the relaxed pause check holds only on the verdict-in-effect admit.
- A `resume` attempt over a run moved to a different row, moved by a goto and then stepped, or a `resume` with no goto over a moved run, is refused as moved, as today.
- An admitted re-launch under a hub pause holds before stepping, as any step does.
- The tackler's consume admits and refuses the same cases as the arbiter's.
- The arbiter verb refuses a blocked Merge-In or Publish halt whose verify-failure record names a target, naming the hand-back; a blocked stop at another row stays eligible.
- A blocked Merge-In stop of another cause, a merge conflict or a dirty parent, while an earlier verify's failure record naming a target is still on disk, stays eligible.
- A blocked Publish stop of another cause, a rejected push or a tree dirtied after the verify, while an earlier verify's failure record naming a target is still on disk, is eligible.
- The arbiter stencil and the alignment stencil name `lyx board get` with a JSON slug payload.
- Record-batch fills the digest's deviation list from the guard, deduplicated by file; a batch whose changes stay inside its targets gets an empty list.
- A changed file with no symbols in the delta is listed unless a card's file target names it or it lies directly in a package directory a card's symbol or package target names.
- An import-only change in a file directly in the package directory of a card's symbol target is not listed; the same change in a file of a subdirectory of that package directory is listed.
- An import-only change in a `_test.go` file of package `<dir>_test`, under a card whose symbol target is in that external test package, is not listed.
- The scope findings appear in the digest and not also in record-batch's warnings; a delta failure leaves the guard-could-not-run warning and no list.
- A fork report carrying `deviations` parses and the key is ignored; one without it parses.
- The implementer stencil asks for no `deviations`; the Master stencil and the Master's summary narrative read the digest's list.
- The alignment stencil escalates only on a design conflict or an open question, records the landing order (or a tie or unknown) for every overlap, and never escalates a text-only overlap; the shared-bookkeeping-file exemption stays.
- The alignment stencil takes the landing order from `lyx loom peers`' `row` and `own_row`, never from a recipe file, and in a hub with no loomyard checkout the fields give each loom row's position in the embedded recipe and 0 for a status whose producer is not a loom row; a darn peer at Publish gets loom's Publish position and one at Darn-Write gets 0.
- A landed parent commit that changes the same behaviour as this run's decision record in a different way, or already makes one of its decisions, escalates as a design conflict; one that changes the same file at a different place does not.

## Notes for the plan writer

- The consume admits live in `internal/loomcli/arbiterconsume.go` and `internal/loomcli/tacklerconsume.go`; `shedengine.Relaunch` applies the target after `Admit`.
- The Publish refusal is `PublishHaltNamesTarget` in `internal/loomcli/arbitereligibility.go`, refusal-spec row "arbiter: stop not eligible, Publish halt naming a target".
- The scope findings enter record-batch's warnings in `internal/websterengine/recordbatch.go`; the guard is `internal/planglyph/scope.go`, its finding type `planindex.Finding`; the digest is `internal/websterengine/digest.go`, its prompt line `digestSummaryLine` in `beginbatch.go`; `ParseReport` decodes with known fields only.
- The peers line is `peerLine` in `internal/loomcli/peers.go`; the embedded loom recipe is `recipes.LoomRecipe` in `contracts/recipes/recipes.go`.
- Stencils: `contracts/stencils/webster/webster-body-implementer.md`, `webster-template-master.md`, `contracts/stencils/loom/loom-template-arbiter.md`, `loom-template-alignment.md`; `contracts/stencils/claims_test.go` pins stencil phrases.
