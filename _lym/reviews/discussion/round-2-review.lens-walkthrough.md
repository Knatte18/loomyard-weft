```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism step by step on one concrete case and checked the end state against the decision's stated purpose.

**Decisions walked**
- D1, `darn-recipe.yaml` with Record-Write added and `verify_source: tree_scope`: there is one record row (RecordWrite), the build segment `{Darn-Write}` comes after it on the `on_done` path, and Finalize's only `on_done` successor is the terminal Friction-Reflect. Purpose met.
- D2, a pawl run started by batten's Run-Shed with `lym loom start`: it passes the named exception, and `ArmAt` is untouched. Purpose met.
- D3, `validate-plan` on a darn run: the recipe has no `Plan-Write` row, so the verb refuses. Purpose met.
- D4, this task's own board body (inline code only, so the longest backtick run is 1): the rule gives a fence of 2 backticks, which is not a fence. Purpose missed (F1).
- D4, a crash in the middle of writing the record: the torn record is kept and committed. Purpose missed (F2).
- D4, an in-flight darn run parked on Drift-Baseline and resumed without the goto: the drift agent is told to read a record that does not exist. Purpose partly met (F5).
- D4, an in-flight darn run with commits, on the route goto, `lym loom step`, goto, resume: `lym loom step` exists. Purpose met.
- D5, the first Discussion-Write gate on a `tapestry` run with the history check: the committed history is empty, so the check passes. Purpose met.
- D6, a no-parent outdated stop on a hub with tree scope off: the verify gate runs the `darn-write` key on the base tree; a pass, or a Stuck once the gate is exhausted, both reach the marker read and halt with the description. Purpose met.
- D6/D7, a no-parent stop answered later by a `Decision for <slug>:` note and a plain resume: the answer never reaches the record. Purpose missed (F3).
- D7, a darn Code-Review fasit of `decision-record.md` alone, holding the fenced body plus the added entries. Met, given D4 works.
- D8, Describe on darn, where the plan record is absent: it renders `(none)`. Purpose met.
- D9, a Finalize reroute to Parent-Align: Drift-Verify, then Code-Bouncer, then Describe. Purpose met.
- D10, a run parked on Tapestry-Preflight: the retired-row map sends it to Preflight, which loads `batcher.yaml` because the recipe has an Implementer row. Purpose met.
- D11, a seed recording `beam`: `lym shed` refuses it as unknown and `lym loom` refuses it as a non-loom recipe. Purpose met.
- D12, a profile with `loom.plan_max_bounces: 2`: `moveRunSettings` moves it to `tapestry`, and the tapestry load resolves it against loom's `review_max_bounces`. Purpose met.
- D13, a profile with `{"loom":{"driver_kind":"go"}}` going through `PrepareRunSettings`: the key can be dropped before the new check sees it. Purpose missed (F4).
- D14, the verbs as they are today. Purpose met.

**Findings**

**F1. BLOCKING, design, D4 (fence form).** The fence rule "one longer than the longest backtick run in the body" has no floor of three, so for most bodies it produces no fence at all.
- Walking it on this task's own body: the longest run is 1, so the fence is ` `` `. A body with no backticks gets a 1-backtick fence.
- Neither is a fence under CommonMark, which needs at least three, nor under `fencedLines`, which tests `HasPrefix("```")`.
- So every body line stays unfenced. A body line `## Decisions` is then the first unfenced `## Decisions` that `decisionsInsertOffset` meets, and `decision add` inserts into the body under `## Goal`. The same applies to `locateOpenRisks` for a body `## Open risks`.
- This defeats D4's own purpose and its acceptance row.
- Fix: the fence length is max(3, longest run + 1).
- Evidence: `internal/discussionparser/append.go:307-345, 349-361`, `carryover.go:234-255`, and the board body (inline code only, no fences).

**F2. MEDIUM, design, D4 (crash during a write).** "Never overwrites one that exists" turns a torn write into a permanent stall, because the record states no atomic write.
- A crash in the middle of writing the record leaves a truncated file. A record cut before `## Acceptance criteria` is one example.
- On resume, Record-Write sees that the file exists, keeps it, and commits it.
- Darn-Write's validate-only `discussion` gate (attempts 3) then reports a missing section and re-prompts a writer that D6 says never edits the record.
- The gate is exhausted, the row returns Stuck, and the way forward is `lym loom resume` (`darnResumeWayForward`). That loops on the same torn record.
- Fix: write each file atomically, to a temporary file and then a rename, or validate an existing file before keeping it.
- Evidence: `internal/loomshed/darnwrite.go:15, 93-95`, `internal/discussionparser/validate.go:65-95`, `darn-recipe.yaml` (Darn-Write gates).

**F3. MEDIUM, design, D6 with D7 (no-parent `Decision for <slug>:` note).** The record does not say what the writer does when, with no parent, a `Decision for <slug>:` note answers the stop.
- Today's stencil has the writer write the note's answer into the record. D6 forbids that ("It never edits the record").
- D6 names only `lym loom decision add` as the way forward.
- An operator can still answer by writing the note and resuming, which is today's stencil route. The resumed writer reads the note (D6 has it read those notes) and follows it, but the answer never enters the record.
- D7's Code-Review perch measures against the record alone, and the board-entry snapshot is gone, so it judges the change against a scope that lacks the answer.
- Fix: state the writer's behavior on an answering note when it has no parent. Either it stops again until `decision add` records the answer, or the note is read as part of the scope by the perch and Drift.
- Evidence: `contracts/stencils/darn/darn-template-write.md` (Step 3 no-parent bullets), and D6 and D7 in the record.

**F4. MEDIUM, design, D13 (where the refusal sits in `PrepareRunSettings`).** The record places the new check "before ValidateLocal" but does not order it against the module's `Migrate`, which `PrepareRunSettings` runs first.
- Reconcile has no generic `RetiredKeys` prune: no hit in `configsync` or `hubreconcile`. So the prune D13 relies on ("until the reconcile prunes it") goes through loom's and darn's Migrate drop lists. For loom that is `MigrateConfig`'s `DropKeys(retiredLoomKeys)`.
- The walk: `PrepareRunSettings` runs `moveRunSettings`, then `module.Migrate` (which drops `driver_kind`), then `checkLocalDocument`.
- A check sitting just before ValidateLocal therefore never sees the key. The profile's `driver_kind: go` is dropped silently, which is exactly what the rationale says refusing prevents.
- Fix: state that the check runs before Migrate, on the raw module object.
- Evidence: `internal/configreg/runsettings.go:45-87`, `internal/loomengine/config.go:660, 709`, `internal/configengine/layers.go:200-230`.

**F5. LOW, design, D4 (in-flight run resumed without the goto).** A darn run parked on Drift-Baseline and resumed without `goto --to Record-Write` runs Drift-Baseline with `subject: decision-record` while no record exists.
- `subjectFields.reading` still tells the agent to read the missing path.
- The record bounds this only at Darn-Write. Any Stuck from Drift-Baseline carries drift's own ways forward, not `goto --to Record-Write`.
- Fix: either have Drift-Baseline refuse a missing record, naming `goto --to Record-Write`, or state that the Darn-Write refusal is the accepted bound.
- Evidence: `internal/driftengine/seats.go:60-63`, and D4's in-flight bullets.

**Tool calls**
1. `lym loom --help` and a grep for retired-key pruning in `configsync`, `hubreconcile` and `configengine`. My context lacked whether `lym loom step` exists (needed to walk D4's route) and whether reconcile prunes `RetiredKeys` generically (needed to walk D13's order).
