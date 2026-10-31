```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's bounds, guarantees and claims about existing code.

**Claims attacked**

- D4, fence length "one longer than the longest backtick run": **broken**. There is no floor of three.
- D4, "the discussion spec reads no fenced line, so `decision add`, the carry-over and the PATTERN parse act only on the record's own sections": **held**, given a valid fence. `missingSections` is the exception, covered in the D5 guard finding.
- D4, a resume after a crash between the writes or before the commit completes the pair: **broken** for a torn write.
- D4, the no-commit way forward "captures the baseline again with the record in place": **broken**.
- D4, Record-Write's commit lands in the weft repository, so Drift-Baseline's clean-tree precondition still holds: **held** (not attacked further).
- D4 and D5, an empty-sections record passes and an empty `## Decisions` has no id to miss: **held** (`decisionids.go:31-33`).
- D5, Darn-Write's `discussion` gate is "a validate-only guard that the writer left the record intact": **broken**.
- D6, a marker present halts after a Done handoff and after a gate-failed Stuck alike, and the halt lets nothing through: **held**. A crash between the session's return and the marker read leads to a respawn that stops again.
- D10, four `CheckSeed` verdicts are pre-empted by Shed's step-1 read: **held** (`seed.go:33-43`).
- D10, `waivesHalfFinished` waived a goto re-entry: **held** (`tapestrypreflight.go:139-158`).
- D10, a repeated `lym loom start` never rewrites a seed: **held** (`sharedbootstrap.go:70`, `:118`, `:176`).
- D3, the off-machine peer falls back to pawl for a project entry, else the entry's `recipe` field: **held** (`battencli/wire.go:523-528`).
- D11, `lym shed` refuses `beam` as unknown and names the available recipes: **held** (`shedrun/seed.go:97`, `shedcli/table.go:80`).
- D12, a profile or `run_settings` naming the old module is translated on read, with nothing rewritten: **held** (`runsettings.go:46`, `keymove.go:63-110`).
- D13, a `CategoryNone` retired key is admitted and dropped silently: **held** (`configengine/doc.go:76`).
- D13, refusing rather than ignoring keeps a pinned run from silently running another driver: **broken** on the hub base layer.
- D13, the refusal check runs in `PrepareRunSettings` before `ValidateLocal`: **broken** ordering. `Migrate` runs earlier still.

**Findings**

1. **BLOCKING, design, D4 (fence form).** The fence is one backtick longer than the longest backtick run in the body, with no minimum of three.
   - Counterexample: this task's own board body has only inline code (runs of 1), so its "fence" is ` `` `. That is not a fence under CommonMark, and `fencedLines` does not see it either: it checks `HasPrefix(trimmed, "```")` (`internal/discussionparser/append.go:349-361`).
   - A body with no backticks gets a 1-tick "fence".
   - Then a body holding `## Decisions` is read by `decisionsInsertOffset` (`append.go:307-347`) as the first `## Decisions` outside a fence. `decision add` lands inside the body, and `CheckDecisionIDs` reads the body's `### D…` headings.
   - D4's own acceptance row (a body holding `## Decisions` and `## Open risks`) fails for any body without a run of at least two backticks.
   - Fix: the fence is `max(3, longest run + 1)` backticks.

2. **MEDIUM, design, D4 (resume after crash).** "Writes whichever of the two files is missing and never overwrites one that exists" does not cover a torn write.
   - Counterexample: lym dies partway through writing `decision-record.md`, leaving a truncated file. On resume Record-Write sees the file exists, keeps it and commits it.
   - Darn-Write's `discussion` gate (validate-only, D5) then fails. Its re-prompt goes to a writer that D6 forbids to edit the record. The gate exhausts and the run halts with `darnResumeWayForward` ("run lym loom resume", `internal/loomshed/darnwrite.go:15`). Every resume repeats the same failure.
   - The record never states that each file is written atomically (temp file plus rename), nor a way forward for a malformed existing file.

3. **MEDIUM, design, D4 (in-flight way forward with no commits).** The record claims that `goto --to Record-Write` then `resume` "captures the baseline again with the record in place of one an agent took with no record to read". The existing code contradicts this:
   - `internal/driftengine/doc.go:68` says DriftBaseline records the scope only "when either is missing", and spawns the agent "with a note that the scope is already recorded when it is".
   - Line 69 says "a set awaited_at and a passed capture take no capture".
   - On the second pass the scope record from the first pass stands, and no new capture runs.
   - The premise behind the no-commit route, and behind choosing it over the step route, is false. Either state that the earlier scope stands (and why that is fine), or name how the scope gets re-recorded.

4. **MEDIUM, design, D5 (Darn-Write's validate-only guard).** The guard claims to prove "the writer left the record intact", but its section check reads fenced lines.
   - `missingSections` (`internal/discussionparser/validate.go:113-129`) matches every line verbatim with no `fencedLines`, despite its own doc.
   - Counterexample: the body holds `## Open risks`, which is D4's own acceptance case. The writer deletes the record's real `## Open risks` heading, and `Validate` still passes on the fenced copy.
   - The Code-Review Bouncer's carry-over then fails: `locateOpenRisks` skips fenced lines and returns `ErrNoOpenRisksHeading` (`carryover.go:234-256`).
   - Bound it: `missingSections` reads headings outside fences only.

5. **MEDIUM, design, D13 (refuse rather than ignore).** D13's rationale says refusing "keeps a run pinned to a driver from silently running another". It refuses only in profiles, `run_settings` and a pair's local files.
   - D13 itself says a hub base `loom.yaml` or `darn.yaml` value is dropped at every load "until the reconcile prunes it", and the reconcile prunes it without moving it.
   - Counterexample: a hub whose base `loom.yaml` tuned `driver_kind: go` (or `darn.yaml` tuned `arbiter`) silently runs `shed.yaml`'s value after landing, and the reconcile then deletes the tuned value.
   - The record neither states this as a bound nor moves the hub-tuned value under `shed`. The hub reviewed here holds none of these keys (grep of `_board/_lym/config`), so the impact is on other hubs.

6. **MEDIUM, design, D13 (where the check runs).** D13 places its refusal "before" `ValidateLocal` in `PrepareRunSettings`, but `PrepareRunSettings` runs `module.Migrate` earlier still (`internal/configreg/runsettings.go:71-83`, then `checkLocalDocument` at :135).
   - Loom's `Migrate` is `MigrateConfig`, which ends with `yamlengine.DropKeys(migrated, retiredLoomKeys)` (`internal/loomengine/config.go:709`).
   - The record leaves the reconcile's prune route for these keys unnamed. If the implementer prunes them the way loom prunes its other retired keys, by adding them to `retiredLoomKeys`, a profile or `run_settings` key is dropped silently before the check sees it.
   - The record must place the check before `Migrate`, or name the prune route.

**Tool calls**

1. `tapestrypreflight.go` `waivesHalfFinished`, the `WriteSeed` refusal in `sharedbootstrap.go`, and `boardRecipe` in `battencli/wire.go`, to check the D10 and D3 claims; the context had only function indexes.
2. Lines of `driftengine/doc.go` on re-capture, plus `lym loom --help`, to check D4's baseline re-capture claim and that `step` exists; the context lacked the Drift-Baseline re-entry doc.
3. `driftengine/doc.go` lines 64-75, the DriftBaseline re-entry rule in full.
4. `checkLocalDocument`, `CheckDecisionIDs`' head, and the unknown-recipe refusals, for the D13 ordering, D5 empty-sections and D11 claims; the context lacked these bodies.
