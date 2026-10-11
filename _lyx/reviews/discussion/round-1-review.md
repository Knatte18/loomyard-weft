---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/discussion/decision-record.md D8
    summary: D8's pre-rename check compares the disk bytes with the bytes it spliced, which differ from the disk by construction, so read literally every carry-over write refuses; A5 and the acceptance criteria mean the bytes it originally read
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D8, R2
    summary: D8's bound and R2 claim the only gap is between the pre-rename compare and the rename, but a writer that read before the rename and writes after the post-rename re-read still clobbers the carry-over silently; R2 also says such a writer renames, while lyx loom decision add writes in place
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/discussion/decision-record.md D4
    summary: D4 clears the marker only in batten's pre-run, but battenPreStep seeds a fresh batten status too, so a fresh run started by lyx batten step keeps a stale marker and opens no window
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4 Rationale
    summary: D4's bound says a run with no batten status file has opened no window yet, which is false when the batten status vanishes mid-run while the slug-scoped scratch marker survives; the second window this task fixes for the child then reappears for batten
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D5
    summary: D5 does not say what a floor hook does with a malformed or non-numeric stored value, and a Migrate error aborts lyx config reconcile for every module, so the choice bounds what the hook may break
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5, Acceptance criteria
    summary: D5 says the rest of the file stays byte-identical as batcher.MigrateConfig does, but MigrateConfig re-encodes the whole document through a yaml encoder, and reconcile writes yamlengine.Reconcile's template-shaped merge rather than the hook's bytes, so the every-other-byte-unchanged criterion rests on a false premise
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D6
    summary: D6 says the dry run prints the same envelope fields as an applied run and that --apply restores exactly today's behaviour, yet board retire-legacy's applied envelope today carries no field and its dry run is to list the legacy files
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D2
    summary: D2 moves the spec's own back-link to the new name, but final-summary-spec.md has no link to itself; its only cross-link points at webster-spec.md, which D2 already lists
  - id: F9
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md D2 Rationale
    summary: D2 Rationale's last sentence restates A1 instead of leaving the departure to the assumption that records it
---

### [BLOCKING] D8's pre-rename compare names the wrong bytes

**Location:** `_lyx/discussion/decision-record.md`, D8 Decision, first sentence.

**Issue:** D8 says `WriteCarryOver` "refuses, writing nothing, when the bytes differ from the ones it spliced".
In `internal/discussionparser/carryover.go`, `writeCarryOver` reads `prior` with `os.ReadFile`, builds `next` by splicing the rendered entry into `prior`'s lines, and renames `next` into place.
The spliced bytes are `next`, which differs from the disk copy whenever the write changes anything, so a check built to the Decision's wording refuses every non-trivial carry-over.
A5 ("compares whole-file bytes") and the acceptance criterion "when the record changes between its read and its rename" both mean `prior`, the bytes the function read.
The Decision is the plan writer's authority, so the three statements disagree on the one point that decides whether the check works.

**Fix:** Say in D8 that the pre-rename re-read is compared with the bytes `WriteCarryOver` read at its start, and word A5 and the acceptance criterion the same way.

### [MEDIUM] D8's race bound omits the window after the post-rename re-read

**Location:** `_lyx/discussion/decision-record.md`, D8 Rationale bound and R2.

**Issue:** `AppendDecision` (`internal/discussionparser/append.go`) reads the record, then writes it back with `os.WriteFile`.
If it reads before the carry-over's rename and writes after the carry-over's post-rename re-read, both checks pass and its write drops the carry-over entry with no error.
So the bound "the checks only turn a silent lost update into an error" overclaims, and R2 names only the window between the pre-rename compare and the rename.
R2 also says a writer in that window is caught "only if it renames after the carry-over", but `lyx loom decision add` never renames: it truncates and rewrites in place.

**Fix:** State in D8's bound and in R2 both windows a lock-free check leaves open: a writer landing between the compare and the rename, and a read-modify-write whose write lands after the post-rename re-read.
Describe the second writer as an in-place write, not a rename.

### [MEDIUM] D4 misses batten's second fresh-status seed site

**Location:** `_lyx/discussion/decision-record.md`, D4 Decision; Scope In names only "the batten fresh-run path in `internal/battencli`".

**Issue:** D4 clears the marker in "the fresh-start branch of batten's pre-run".
`internal/battencli/arm.go` seeds a fresh batten status in two places: `battenPreRun` and `battenPreStepUnmarked`, which seeds when absent so a fresh slug's first `lyx batten step` does not loop.
A fresh run started through `lyx batten step` then never clears a marker left in the slug-scoped `.lyx` scratch directory by an earlier run of the same slug, so its first spawn opens no window.
That contradicts D4's own rule that the marker resets once per batten run.

**Fix:** Clear the marker at every point batten seeds a fresh status, or name the single seam both hooks share, and extend the acceptance criterion to a fresh run started by `lyx batten step`.

### [MEDIUM] D4's bound does not hold when the batten status vanishes

**Location:** `_lyx/discussion/decision-record.md`, D4 Rationale, "Bound: the reset can only clear the marker of a run with no batten status file, which has opened no window yet."

**Issue:** The batten status lives under the durable `_lyx` run directory (`shedrun.RunDir`), while the marker lives under the ephemeral `.lyx` scratch directory (`shedrun.ScratchDir`).
The two can diverge: the fault this task fixes is a child's status file vanishing mid-run while its marker stays.
If the batten status file vanishes the same way, the next `lyx batten run` or `step` seeds a fresh status, clears the marker and opens a second window for a pair whose first window is still attached.
So "has opened no window yet" is an assumption, not a bound, and the decision record states it as fact.

**Fix:** Either state that a vanished batten status opens a second window and record it under Open risks, or make the bound conditional on the batten status never vanishing.

### [MEDIUM] D5 leaves the malformed-value case open

**Location:** `_lyx/discussion/decision-record.md`, D5 Decision.

**Issue:** D5 says what a hook does with a value below, at or above the floor, or an absent key, but not with a value the loader cannot decode, such as `poll_interval_ms: fast`.
In `internal/configsync/configsync.go`, a `Migrate` error returns a `FileError` from the whole reconcile, so no module reconciles.
`batcher.MigrateConfig` errors on an unparseable document, marked `configengine.ErrInvalid`.
A hook that errors on a non-numeric key therefore turns a file that reconcile handles today into one that blocks every module's reconcile.
A hook that skips the key leaves it to the loader, which fails at load anyway.
Whichever the plan writer picks, it decides what the new hooks may break.

**Fix:** State in D5 whether a hook leaves a non-integer value untouched with no report line, or returns an error, and if it returns one, what the operator sees.

### [MEDIUM] D5's byte-identity premise is false

**Location:** `_lyx/discussion/decision-record.md`, D5 Decision ("the rest of the file stays byte-identical as `batcher.MigrateConfig` does") and the reconcile acceptance criterion ("writes the floor value with every other byte unchanged").

**Issue:** `batcher.MigrateConfig` (`internal/batcher/config.go`) edits a `yaml.Node` tree and re-encodes the whole document with `yaml.NewEncoder` at indent 2, so its output is not byte-identical outside the changed keys.
Its own doc comment promises only that everything else is "carried through yaml node editing with its comments".
`ReconcileAll` also writes `yamlengine.Reconcile`'s merge of template and migrated bytes, not the hook's output.
So the written file has the template's layout, and "every other byte unchanged" holds only for a file already in that layout.
The acceptance criterion then pins a property neither the model hook nor the write path gives.

**Fix:** Drop the batcher comparison, or say what is preserved: every other key and value, with the write being the merge reconcile already produces.
Restate the acceptance criterion as: the applied file equals what reconcile writes for the same input with the key at the floor value.

### [MEDIUM] D6's dry-run envelope does not match retire-legacy's applied envelope

**Location:** `_lyx/discussion/decision-record.md`, D6 Decision and bound.

**Issue:** D6 says the dry run "prints the same envelope fields an applied run would, filled with what it would change", and that `--apply` "restores exactly today's behaviour and nothing more".
In `internal/boardcli/cli.go`, `retire-legacy` today returns `outputSuccess(out)` with no fields, while D6 has its dry run list the legacy files it would fold and delete.
So either the applied run gains a field, contradicting "exactly today's behaviour", or the dry run carries a field the applied run lacks, contradicting "the same envelope fields".
For the other two verbs the claim holds: `board prune` prints `removed`, and `fabric unwire` prints `junctions_removed`, `weft_content` and `git_exclude`.

**Fix:** Name the field `retire-legacy` reports in both modes, and word the bound as today's behaviour plus the `applied` field and that report.

### [LOW] D2 names a self back-link that does not exist

**Location:** `_lyx/discussion/decision-record.md`, D2 Decision, "and the spec's own back-link".

**Issue:** `contracts/specs/final-summary-spec.md` links only `webster-spec.md` and never names its own file.
The links that do name it are in `webster-spec.md`, `docs/overview.md`, the darn and landing stencils and `internal/darnengine/spec_test.go`, all covered by D1, D2 and the "no file references the old name" criterion.
The phrase sends the plan writer looking for an edit that has no site.

**Fix:** Drop "the spec's own back-link" from D2's list.

### [NIT] D2 Rationale restates A1

**Location:** `_lyx/discussion/decision-record.md`, D2 Rationale, last sentence.

**Issue:** "This departs from the board text's literal path; the auto-mode assumption A1 records it." repeats what A1 states in full.
The sentence carries no requirement, so removing it loses nothing for the plan writer.

**Fix:** Delete the sentence.

## Verdicts on focus entries with no finding

- D6, callers outside the lyx-shipped list: D6 states that a bare call exits 0 with `"applied": false` and changes nothing, which is what an operator script or agent prompt sees, and the bound is the read-only dry run.
  Not a defect.
  The envelope comparison for each verb is F7.
- D2, Scope and the board-text departure: the rename, registration and reference move agree with Scope In and the acceptance criteria, and A1 records the departure from the board's literal path.
  The premise holds: `contracts/specs/specs.go` registers only `loom-plan-spec`, and the darn stencil names `{{.specs_dir}}/final-summary-spec.md`.
  Only F8 and F9 remain.
- D5, observed fault: the board's observed fault is shuttle's `poll_interval_ms: 500` under the 1000 floor, which matches D5.
  The shuttle and orch loaders warn and their accessors floor the value, rather than rewriting it in the loader; the effective value is still unchanged, so the bound holds in substance.
- D7, the only-drops bound: `internal/boardcli/cli.go` holds the only `IndexNoteAnnotation` in production code, and every board verb that takes a payload is operator-audience, so naming `operator` drops the note only from the internal index.
  The live-tree test adds no note.
  Not a defect.
- Relocation check: apart from F9, the Rationale text carries reasons or bounds the plan writer needs, and nothing else belongs in `support-log.md`.
