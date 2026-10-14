---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D3, the created-by-an-earlier-card bound)
    summary: D3 counts only Create targets and Rename destinations as creating a directory, so a Verify naming a package that an earlier Custom card or a member-glyph Create target creates is a blocking false positive on a valid plan
    fix: In D3's Decision, define the created-directory set as the directories CardTargetDirs yields for every non-Delete group of each card numbered at or below the Verify's card (the whole plan for the overview chain), which covers member-glyph Create targets and Custom groups, instead of the path-missing create and rename unions; or, if the unions are kept, add a Bound bullet that a Verify on or after a card with a Custom group is not checked, as for Move. Add the matching acceptance rows (a Create card targeting internal/new/x.go#Foo, and a Custom card, each followed by a Verify on ./internal/new, report nothing), and drop the Notes bullet that points D3 at the create and rename unions behind path-missing if the first option is taken.
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/discussion/decision-record.md (Scope In, D5, Constraints)
    summary: D5 adds the status-unreadable cause and a new way forward to a shed loop stop, but neither Scope nor D5 updates the step interrupted row in contracts/specs/refusal-spec.md, which PATTERN-refusal-way-forward requires in the same commit and which already wrongly says a vanished status file is written failed
    fix: In Scope In, extend the contracts/specs/refusal-spec.md bullet to cover the shed section's step interrupted row as well as the gate table. In D5's Decision, add a bullet that this row names the status-missing and status-unreadable causes, states that at these causes the status file is left as found rather than written failed, and gives D5's way forward. Name the row in the PATTERN-refusal-way-forward Constraints bullet.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D5, way forward and Bound)
    summary: D5's bound does not say what a downstream reader or resume does with the status left as found, and its single way forward dead-ends at status-missing, since re-running step --until-stop on an absent file meets the status file missing refusal, while an unreadable-to-this-binary file can later read running so lyx loom resume with a live driver answers already running
    fix: Split D5's way forward by cause. For status-missing, name the recipe's own reseed route that refusal-spec's status file missing row gives (lyx loom start for loom, lyx batten run <slug> for batten, lyx shed seed otherwise). For status-unreadable, keep the re-run of lyx shed step <run> --until-stop once the file reads. Add to D5's Bound one sentence that a file that later reads running is resumed by that re-step, not by lyx loom resume, which reports already running while a driver is live, and that lyx loom arbiter refuses either stop under its status-unreadable row.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (D5, No re-step bullet)
    summary: The no-re-step rationale rests on atomic writes, but readStatusAt also reports false on a read-lock acquisition error and a raw read error, which are transient, so status-unreadable covers more than a decode error
    fix: In D5's No re-step bullet, either restrict status-unreadable to a decode error and leave a lock or raw read error on the transient path with one re-step, or state in D5's Bound that a lock or raw read error also stops as status-unreadable with no re-step and the way forward's re-run covers it.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (Acceptance criteria)
    summary: D2 and D7 have no criterion that fails when they are not implemented, since lyx loom lint-pattern passes on the unchanged text, and the unknown-key criterion tests e5b56fe91's behaviour, which no Decision states
    fix: Replace the lint-pattern criterion with criteria that read the changed text, for example PATTERN.md's PATTERN-glyph-conversion-chokepoint line names planparser's external-test unit mapping, pattern/PATTERN-cli-cobra.md's audience bullet states the runnable-group rule, and the Plan-Review rubric carries a CLI-shape check pointing at that bullet, keeping lint-pattern as a further condition. Delete the criterion that a status.json carrying an unknown key is read normally, or add a Decision that requires it.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (D5, Premise second sentence)
    summary: D5's premise sentence attributing the #581 halt to an older-binary strict decode is deliberation narrative carrying no requirement, already recorded in support-log.md Turn 4, Q5 and the Auto-mode assumptions
    fix: Delete the Premise sentence beginning The #581 halt most likely came from from D5 in decision-record.md, leaving the first Premise sentence, since support-log.md Turn 4 and Q5 already hold it and the Auto-mode assumption names the inference.
---

### [BLOCKING] D3's created-directory bound misses Custom cards and member-glyph Create targets

**Location:** `_lyx/discussion/decision-record.md`, D3 Decision and Notes for the plan writer.

**Issue:** D3 reports a relative package argument whose directory neither exists nor "holds a Create target or Rename destination" of an earlier card, and the Notes point it at the create and rename unions behind `path-missing`.
Those unions are built through `diskPathForRef` (`internal/planparser/shape.go`), which maps only path refs and self glyphs: a member glyph such as `internal/new/x.go#Foo` maps to nothing, and `createTargetsUnion` (`internal/planparser/validate.go`) reads only `Create` groups, never `Custom`, the plan spec's explicit escape hatch.
A plan whose card 2 creates `internal/new` through a member-glyph Create target or a Custom group, and whose card 3 Verify is `lyx gate test ./internal/new`, is valid, yet D3 reports it, the finding is blocking, and its way forward ("correct the Verify line") is wrong for it.
The Move bound shows the writer saw this class of gap; Custom and member-glyph Create are the same class and are unbounded.
The overview chain, checked against the whole plan, has the same hole.

### [BLOCKING] D5's changed stop has no refusal-spec row in scope

**Location:** `_lyx/discussion/decision-record.md`, Scope In, D5 and Constraints.

**Issue:** shed is a module `PATTERN-refusal-way-forward` binds, and its section's `step interrupted` row in `contracts/specs/refusal-spec.md` covers the loop stop D5 changes ("or the status file vanished; … the status file is written `failed`", way forward the generic re-run).
D5 adds the cause `status-unreadable: <error>`, keeps the file as found and gives a new way forward, so the row is wrong after D5 lands, and the PATTERN entry requires the row in the same commit.
Scope In names `refusal-spec.md` only for the `lyx gate test` table, and the Constraints bullet for `PATTERN-refusal-way-forward` names only the way forward's runnability.

### [MEDIUM] D5's bound leaves resume and the reseed path unstated

**Location:** `_lyx/discussion/decision-record.md`, D5 way-forward bullet and Bound.

**Issue:** The premise is sound: `readStatusAt` (`internal/shedverbs/loop.go`) returns false for `!found` and for any error from `state.ReadJSONNoMkdir`, which covers absent, read and decode failures.
What the bound leaves out is the reader after the stop.
At `status-missing` the way forward is "re-running `lyx shed step <run> --until-stop` once the file reads", but nothing makes an absent file read again: `step` refuses with refusal-spec's `status file missing` row, whose way forward is the recipe's own reseed.
At `status-unreadable` from an older binary's strict decode, the case D5 itself names as #581's likely cause, the current binary reads the file as `running`; `internal/loomcli/resume.go` then answers "the run is already running" when the driver is live, which is the outcome `PATTERN-failure-stop-visible` exists to prevent, and `lyx loom arbiter` refuses under its `status or envelope unreadable` row.
The departure is stated, but not what it leaves a downstream reader or resume to do.

### [LOW] D5's no-re-step rationale covers decode only

**Location:** `_lyx/discussion/decision-record.md`, D5 Decision, No re-step bullet.

**Issue:** `state.ReadJSONNoMkdir` (`internal/state/state.go`) also returns an error when it cannot acquire the read lock and on a non-ENOENT read error, both of which `readStatusAt` folds into false.
Atomic rename rules out a half-written file, not those, so `status-unreadable` with no re-step stops a run on a transient fault the shed loop otherwise re-steps once.

### [LOW] Acceptance criteria miss D2 and D7 and test an unstated behaviour

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria.

**Issue:** D1, D3, D4 and D5 each have a criterion that fails without them.
D2 and D7 have only "`lyx loom lint-pattern` passes with D2's and D7's PATTERN text", which passes on the current text too; D7's background, `docs/overview.md` and rubric edits have no criterion at all.
"A `status.json` carrying an unknown key is read normally and the loop steps" pins `e5b56fe91`'s behaviour, already in the base, which no Decision states.
D6 needs no criterion, since it changes no code.

### [LOW] D5's root-cause sentence belongs in the support log

**Location:** `_lyx/discussion/decision-record.md`, D5 Premise.

**Issue:** The sentence attributing #581 to an older-binary strict decode of `routed_to` carries no requirement: D5's Decision holds whatever the cause, as the Auto-mode assumption says.
Per the completeness-before-leanness test there is nothing to extract; support-log.md Turn 4 and Q5 already hold the narrative.

## Rejected

- D1's bound on a real `*_test` directory: it states the mapping is disk-free and unconditional and that such a directory maps to its sibling; `find` over the worktree finds no directory named `*_test`, so the bound and the Auto-mode assumption hold.
- D2's chokepoint exception: it is bounded to the one helper and the external-test pseudo-unit, and `vocabname.go`'s `strings.TrimSuffix(unit, "_test")` is the existing precedent it absorbs.
- D4's attack surface: the bound names what passes unchanged (import paths, absolute paths, patterns, `.go` files) and that a refused request runs no package; the gate table in refusal-spec.md exists and Scope names it.
- D6's evidence: `78e31aa06` (the bounded sweep retry) is dated 11:27Z, before #584, and `lyx gate test ./internal/lyxtemp -- -race -run TestSweep -count=1000` passed when re-run for this review; Goal ("settle") and Scope stay consistent with a no-change outcome.
- D2's account of `vocabname.go` in decision-record.md: one clause, and it carries the requirement that the existing trim is folded into the sanctioned site, so it stays.
