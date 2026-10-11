---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5
    summary: D5 says the loader rejects a non-integer stored value, but the loader resolves env markers and reads a null as 0, raising both to the floor, so the non-integer rule rests on a false premise and leaves null undecided
  - id: F2
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D6
    summary: D6 claims to match fabric's --apply idiom but does not decide whether bare fabric unwire joins FabricReaderVerbs and the role guard's readerVerb, as bare prune and cleanup do
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md R4
    summary: R4 says a vanished batten status always opens a second window, but InnerRun opens one only after a spawn, so an adopted running child opens none
---

### [MEDIUM] D5's non-integer rule rests on a false premise about the loader

**Location:** `_lyx/discussion/decision-record.md`, D5, the sentence beginning "A key whose value does not decode as an integer"; the matching acceptance criterion.

**Issue:** D5 says a key whose value does not decode as an integer is left untouched because "the module's loader rejects that value at load, as it does today".
That holds for a value such as `abc`, but not for two non-integer shapes the loaders accept:

- An env marker, `${env:NAME}` or `${env:NAME:-500}`.
  `configengine.load` runs `yamlengine.Resolve` over the file before the engine unmarshals it (`internal/configengine/config.go`, `internal/yamlengine/resolve.go`), so the loader reads the resolved number.
  If that number is below the floor, `shuttleengine.LoadConfig`, `orchengine`'s loader and `loomengine`'s `step_idle_timeout_min` check raise it and warn on every load.
  Leaving the marker untouched is right, since rewriting it would discard the operator's env indirection, but the stated reason is false and the warning the task targets stays.
- A null value, `poll_interval_ms:` or `~`.
  `yamlengine.MissingKeys` counts a key with an empty value as present, and yaml.v3 decodes a null into an `int` field as 0 without error, so the loader accepts it and raises 0 to the floor.
  Whether D5's hook treats null as "does not decode as an integer" (left untouched, so the warning stays) or as 0 (raised, matching the loader) depends on how the hook decodes the node, and D5 does not say.

The D5 bound ("a hook only raises a value its own loader already raises") still holds in both cases.
The defect is the false premise and the undecided null case.

**Fix:** Replace the premise with the actual loader behaviour.
State per shape what the hook does: an env-marker scalar is left untouched with no report line, because its value is resolved per environment, and the loader may still raise it.
A null is either raised to the floor like the loader's 0, or left untouched, but D5 must say which.
A value that does not decode at all, such as `abc`, is left untouched and the loader rejects it.
Name the null and env-marker cases in the acceptance criterion's "yields no line" clause, or in a raise clause, to match.

### [LOW] D6 leaves bare `fabric unwire`'s reader classification undecided

**Location:** `_lyx/discussion/decision-record.md`, D6; `internal/fabricengine/refscanner.go` `FabricReaderVerbs`; `internal/fabriccli/roleguard.go` `readerVerb`.

**Issue:** D6's rationale is that the dry-run default "matches fabric's existing `--apply` idiom".
In fabric today, that idiom has a second part: `prune` and `cleanup` without `--apply` are in `FabricReaderVerbs`, and `readerVerb` lets a sandboxed task session run them because they only read.
After D6, bare `fabric unwire` is read-only too, but D6 does not say whether it joins that set.
If it stays out, the role guard keeps refusing the harmless dry run from a task session, and webster's audit keeps classing it as a mutating fabric reference.
If it joins, the attack-surface question applies, and the bound is that only an invocation without `--apply` passes, as for `prune` and `cleanup`.
Either choice is defensible, but the plan writer should not have to make it.

**Fix:** Add one sentence to D6 that makes the choice.
Either bare `unwire` joins `FabricReaderVerbs`, with `readerVerb` passing it only without `--apply`, or it stays out of the reader set, which keeps the guard stricter than the idiom D6 cites, and the sentence gives the reason.

### [LOW] R4 overstates when a second terminal window opens

**Location:** `_lyx/discussion/decision-record.md`, R4.

**Issue:** R4 says a vanished batten status "makes the next `lyx batten run` or `step` seed a fresh status, clear the marker and open a second window".
The seed and the clear follow from D4.
The window does not always open: InnerRun's `Call` (`internal/battenshed/innerrun.go`) calls `openTerminalOnce` only after a successful spawn.
A child whose status is found running with its driver at work is adopted without a spawn, and no window opens.
D4's own bound is worded correctly ("it clears the marker of a window already open only when…"), so R4 and D4 disagree on the consequence.

**Fix:** Reword R4 so a second window opens only when InnerRun then spawns the child, and say the marker is cleared either way.

## Focus verdicts

- D8: D8 ("the bytes differ from the ones it read at its start"), A5 ("whole-file bytes on disk with the bytes it read at its start") and the acceptance criterion ("the bytes on disk before its rename differ from the bytes it read at its start") all name `prior`, the whole file `writeCarryOver` reads first.
  They agree.
  R2's two windows cover every interleaving left open: a write between the pre-rename compare and the rename, and a read-modify-write that reads before the rename and writes after the post-rename re-read.
  The second window also covers `AppendDecision` writing in place with `os.WriteFile`, including its restore-on-findings write, which is a second write from a read taken before the rename.
  A writer that lands between the rename and the re-read is caught, and so is an in-place write that truncates before the pre-rename compare.
  Not a defect.
- D4: `battenPreRun` and `battenPreStepUnmarked` are the only two places that seed a batten status with `CurrentProducer: battenrecipe.NameWorktreeCreate`, and D4 resets the marker at both.
  The bound holds when the durable status vanishes and the ephemeral marker survives.
  The marker is cleared only at that reseed, which batten treats as a new run.
  The overstatement is in R4, not the bound (F3).
- D5: `ReconcileAll` aborts the whole reconcile on a `Migrate` error.
  D5's leave-untouched rule avoids that abort, but its stated premise is false for env markers and null (F1).
  On byte preservation, D5 claims only that the hook's output carries other keys and comments, and it states that the applied write is reconcile's template-shaped merge.
  `yamlengine.Reconcile` copies each existing leaf's `Value`, `Tag` and `Style` onto the template node (`applyExistingOverrides`), so a hook that sets the scalar's `Value` to the floor produces the same merged file as a hand-edited input with the key at the floor.
  The acceptance criterion matches what the write path preserves.
- D6: today, `retire-legacy` prints `{"ok":true}`, `board prune` prints `removed`, and `fabric unwire` prints `junctions_removed`, `weft_content` and `git_exclude` with its mutation record.
  D6's bound and the acceptance criterion list the additions exactly: `applied` on all three, plus `legacy_files` on `retire-legacy`.
  The claim that `--apply` restores today's behaviour is consistent across D6, its bound and the acceptance criteria.
  The open question is reader classification (F2).
