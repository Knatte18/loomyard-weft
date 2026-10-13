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
The pause check refuses only a pause condition present now that the attempt did not record at spawn; a condition the goto cleared does not refuse.
The arbiter's and the tackler's consume share one admit check, so the two cannot drift apart.
`shedengine.Relaunch` learns from its admit hook that the target is already in effect and skips its goto; the signal is generic and names no loom type.
Bound: besides the verdict's own move, this lets through a parent's goto to the same row, which the re-launch then steps without waiting for the parent's resume, as the Go driver's recheck already does for an orch goto.
A step run after the goto, a different row, a resume with no goto and an already-applied attempt are refused as today, under the same status lock.

Rationale: the refusal parked runs for the parent although the run stood where the verdict wanted it, and the parent's only action was the same re-launch by hand.

### D2 — The mover is batten's verify retry, and the arbiter refuses its Merge-In stop too

Decision: the second actor is batten's first-verify-failure retry (`internal/battenshed`'s publish retry), which moves a child blocked at Merge-In or Publish to its failure record's target and resumes it, independent of the child's driver.
The arbiter verb already refuses a `blocked` Publish halt whose verify-failure record names a target, as belonging to batten's retry and then to the parent.
That refusal widens to Merge-In: a `blocked` stop at Merge-In or Publish whose verify-failure record names a target is not eligible.
The existing refusal-spec row is edited to name both rows; no second row is added.
The LLM driver acts on it through its existing arbiter-refusal path and parks; its stencil gains no new rule, and batten's retry, or the parent when no batten retries, takes the run on.
Bound: the refusal removes the arbiter only from a Merge-In verify halt whose failure record names a target, the case Publish already has; every other blocked stop keeps its eligibility.

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
The guard also reports, at file level, a changed file the delta gives no symbols for, unless a card target names that file.
The scope findings no longer also ride record-batch's warnings, so a reader meets them once, in the digest.
When the delta could not be computed, the digest carries no list and the existing guard-could-not-run warning stays.
The Master stencil, the fork prompt's predecessor digest line and the PR narrative name the digest's list as the batch's deviations, with the guard as its source.
The list stays informational: it never makes a batch `stuck` or `FAILED`.

Rationale: a fork cannot map a glyph to its file without the comparison the guard makes, so its list mixed real deviations with the batch's own files; the guard runs over every batch delta, and the file-level comparison keeps the non-code files the fork's list used to carry.

### D5 — The alignment agent settles landing order itself

Decision: an overlap with a peer run or a landed parent commit no longer escalates on its own.
The agent applies the landing-order rule: the run furthest along by its recipe row lands first, and the other takes its changes at Merge-In.
It approves (or rejects on a scope deviation) and records in its report each overlapping peer, what overlaps and the landing order the rule gives; two peers on the same row, or a peer whose status is unreadable or not a loom row, are reported as a tie or an unknown order, and the verdict is unchanged.
It escalates on an overlap only when the designs conflict: both runs change the same behaviour in different ways, a recorded decision in one contradicts or duplicates one in the other, or both change the same PATTERN entry in different ways.
A text-only overlap, the same file or package changed at different places, is never a reason to escalate.
An open question that needs a decision the board and the records do not give still escalates, as today.
The rule applies at both `parent-review` and `pr-gate`.
Bound: the agent gains no new action; it only stops escalating text-only overlaps, and the parent lock and Merge-In already make whichever run lands second take the first's changes.

Rationale: the orch settled every such escalation of 2026-10-12 by this one rule with no design call, and the operator wants the orch woken only for stops that need a parent choice.

## Constraints

- This task builds on `go-driver-assistant`, landed: the tackler consume and the Go driver's verify facts are its work.
- `PATTERN-refusal-way-forward`: D2 edits the existing Publish refusal row in `contracts/specs/refusal-spec.md` and its test; D1 narrows the trigger of the "run moved or pause conditions changed" consume row and the tackler's matching row.
- `PATTERN-shed-producer-seam`: D1's skip signal in `shedengine` stays generic; the verdict-in-effect logic stays in `loomcli`.
- `PATTERN-glyph-conversion-chokepoint`: D4's file-level comparison takes a target's file through `glyph`'s conversions only.
- `PATTERN-stencil-ownership`, `PATTERN-producer-pointer-rule`: the stencil edits stay stencil-store files read at call time; the Master stencil points at the digest, never restates the guard's comparison.
- `PATTERN-tolerant-records`: the digest stays a Go-written JSON record rewritten through `internal/state`; a missing `deviations` field reads as an empty list.
- `PATTERN-parent-lock`: D5 relies on the parent lock and Merge-In for landing order and changes neither.
- New top-level tests stay within each package's row in `cmd/lyx/testdata/test-budget.yaml`, or the card lists that file under `Edit:`.
- Docs land in the same commit: `internal/loomcli/doc.go` (# Arbiter eligibility and consume, # Tackler consume), `internal/websterengine/doc.go` (fork-return contract), `internal/planglyph`'s scope doc, `internal/alignment/doc.go`.

## Auto-mode assumptions

- The incidents' arbiter attempt records are gone, so D2's mover rests on the code path and the incidents' shape.
- The tackler's consume takes D1 too, since it is the same re-launch shape over the same status lock.

## Open risks

- `arbiter-when` later refuses every blocked stop whose envelope names a goto; it extends D2's widened check and its spec row rather than adding another.
- If the mover was a different actor in either incident, D1 still admits the re-launch, but D2 would not have closed that incident's race.
- D4 removes the fork-reported list that `glyph-use-audit` planned to compare against the guard; runs after this lands give that audit question nothing to compare.

## Acceptance criteria

- A `resume` attempt with a goto, over a run moved by a goto to that target after the spawn, is consumed: the attempt is marked applied, the history gains no second goto entry and the loop steps from the target.
- The same case where the attempt recorded a pause condition at spawn that the goto cleared is consumed too; a pause condition present now and absent at spawn is refused as paused.
- A `resume` attempt over a run moved to a different row, moved by a goto and then stepped, or a `resume` with no goto over a moved run, is refused as moved, as today.
- An admitted re-launch under a hub pause holds before stepping, as any step does.
- The tackler's consume admits and refuses the same cases as the arbiter's.
- The arbiter verb refuses a blocked Merge-In or Publish halt whose verify-failure record names a target, naming the hand-back; a blocked stop at another row stays eligible.
- The arbiter stencil and the alignment stencil name `lyx board get` with a JSON slug payload.
- Record-batch fills the digest's deviation list from the guard, deduplicated by file; a batch whose changes stay inside its targets gets an empty list.
- A changed file with no symbols in the delta is listed unless a card target names it.
- The scope findings appear in the digest and not also in record-batch's warnings; a delta failure leaves the guard-could-not-run warning and no list.
- A fork report carrying `deviations` parses and the key is ignored; one without it parses.
- The implementer stencil asks for no `deviations`; the Master stencil and the PR narrative read the digest's list.
- The alignment stencil escalates only on a design conflict or an open question, records the landing order (or a tie or unknown) for every overlap, and never escalates a text-only overlap; the shared-bookkeeping-file exemption stays.

## Notes for the plan writer

- The consume admits live in `internal/loomcli/arbiterconsume.go` and `internal/loomcli/tacklerconsume.go`; `shedengine.Relaunch` applies the target after `Admit`.
- The Publish refusal is `PublishHaltNamesTarget` in `internal/loomcli/arbitereligibility.go`, refusal-spec row "arbiter: stop not eligible, Publish halt naming a target".
- The scope findings enter record-batch's warnings in `internal/websterengine/recordbatch.go`; the guard is `internal/planglyph/scope.go`, its finding type `planindex.Finding`; the digest is `internal/websterengine/digest.go`, its prompt line `digestSummaryLine` in `beginbatch.go`; `ParseReport` decodes with known fields only.
- Stencils: `contracts/stencils/webster/webster-body-implementer.md`, `webster-template-master.md`, `contracts/stencils/loom/loom-template-arbiter.md`, `loom-template-alignment.md`; `contracts/stencils/claims_test.go` pins stencil phrases.
