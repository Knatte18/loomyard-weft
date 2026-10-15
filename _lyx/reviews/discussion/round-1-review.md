---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:D1 (the Intermediate files paragraph)
    summary: D1 clears an operator-supplied --work directory at the start of every run with no bound on what it may delete, so --work naming the repo root, the hub, the home directory, a directory holding the --out report or any unrelated non-empty directory wipes it.
    fix: "In D1, replace clear-at-start with a bounded rule and state it as the Decision's bound: the tool marks a work directory it creates with a marker file (for example `.forkaudit-work`), clears only a directory that is empty or holds that marker, and deletes only its own named intermediate files and subdirectories inside it; it exits 1 naming --work when the directory is non-empty without the marker, when it equals or contains the repo root, the hub or --out's directory, or when --out lies inside it. Add an acceptance criterion: a non-empty --work without the marker and a --work holding --out each exit 1 naming --work and delete nothing; an unwritable --work exits 1 naming it."
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:Constraints (first bullet)
    summary: The script spells _lyx and .lyx paths, which PATTERN-lyxdirs-single-declarer forbids in any production file but internal/lyxdirs; the record declares a departure on its own authority but plans no PATTERN.md change, so the landed code violates PATTERN as written.
    fix: "In Constraints, replace the self-declared departure with a required change: the task amends PATTERN.md's PATTERN-lyxdirs-single-declarer line, in the same commit as the script, to admit tools/forkaudit/forkaudit.sh's head variable block as the one non-Go site that spells those literals, with the reason that a shell tool cannot import internal/lyxdirs. Add PATTERN.md to Scope In. Keep the -weft sentence as it is, since internal/weftname is not a PATTERN entry."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D3 (the trace-value-outside-three bullet)
    summary: D3 makes any trace pack_layout value outside file, inline and none the fork's arm with no bound on what that value may hold, so a value equal to a marker arm (raf, unknown) merges foreign forks into that arm, and a value with a pipe, newline or backtick breaks the report's markdown tables.
    fix: "In D3's bullet, state the bound: a trace value becomes the arm only when it matches [a-z0-9_-]+ and is not one of the marker table's arm names; the arm is then named trace:<value>, so it never merges with a prompt-classified arm. Any other value leaves the fork unknown and is listed once in the samples section as an unrecognized trace value. Extend the matching acceptance criterion with a trace value equal to raf and one holding a pipe."
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D7 (the Batch resolution bullet)
    summary: Batch resolution searches every pair's state.json history with git log --all for a spawnedAt 1 to 6 seconds before the fork's start, so with pairs running concurrently a batch of another run spawned in that window resolves first and gives the fork the wrong cards and startSha, which silently mis-classes every pack-check span and the D3 cross-check.
    fix: "In D7's Batch resolution bullet, restrict the search to the fork's own run: the records branch <run>-weft and the tags archive/<run>/*, where <run> is the fork's project run name from D2; a match outside those refs is not used. Add an acceptance criterion: a records fixture with two runs whose batches share a spawnedAt resolves each fork to its own run's batch."
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:D2 (the --runs bullet and the run-name exit-2 rule)
    summary: D2 does not say whether a --runs name matches a project directory exactly or by prefix, and its character set admits . and .. and names with _ or . that Claude Code's directory encoding turns into -, so run-pane would match run-pane-timing under a prefix rule, .. passes validation and is joined into the hub/run/.lyx/logs trace path, and a name with _ never matches.
    fix: "In D2, state that a run name matches only the project directory whose name equals the hub's encoded path, -, and the run name, exactly; restrict run names to lowercase letters, digits and -, which is the slug character set and survives the encoding unchanged; and drop . and _ from the allowed set. The existing invalid-input criterion then covers .. and _ through D2's list."
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:D7 (the missing --records or --code bullet)
    summary: D7 exits 1 on a missing --code directory when --plan alone is given, but D6's plan section reads only the records repo and transcripts, and plan-refs.sh classifies references by their text, so --plan never reads --code.
    fix: "In D7's bullet, make a missing --code exit 1 only under --pack-check, and a missing --records exit 1 under --pack-check or --plan; extend the acceptance criterion to say --plan without a --code directory succeeds."
  - id: F7
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:D5 (the Rows sort paragraph)
    summary: Rows sort by arm, project and start time only, so two forks of one run with an equal start time fall back to find's filesystem order, which the identical-reports acceptance criterion does not allow.
    fix: "In D5, append the agent id as the last sort key: rows sort by arm, then project, then start time, then agent id."
---

### [BLOCKING] `--work` clearing has no bound

**Location:** `_lyx/discussion/decision-record.md`, D1, the paragraph on intermediate files.
**Issue:** D1 says the work directory is "cleared at the start of each run" and that `--work <dir>` overrides it, with nothing on what clearing may delete.
`--work` is an operator-supplied path, so the Decision introduces a deletion edge over an arbitrary directory, and the discussion spec requires such a Decision to state what it lets through and what bounds it.
Cases with no stated outcome: `--work .`, `--work ~`, `--work` naming the repo root or the hub, a non-empty unrelated directory, and an `--out` inside `--work`, where the clear deletes the previous report and breaks D1's own "a failed run leaves the previous report intact".
The other path flags are bounded in effect: `--projects`, `--hub`, `--records` and `--code` are only read, and `--out` writes one operator-named file through a temporary name, which is normal output-flag behavior.
No acceptance criterion covers the clear or an unwritable `--work`, though D1 names the latter as an exit-1 case.

### [BLOCKING] `_lyx` literals depart from PATTERN without amending it

**Location:** `_lyx/discussion/decision-record.md`, Constraints, first bullet.
**Issue:** `PATTERN-lyxdirs-single-declarer` says no production file but `internal/lyxdirs` spells `_lyx` or `.lyx`, and CLAUDE.md says code under `tools/` follows PATTERN like everything else.
The record calls its literals "a named departure for this one shell tool", but PATTERN has no departure mechanism and the record plans no edit to it, so after landing PATTERN states an invariant the repo breaks.
The script does need the literals; the gap is that the exception has to land in PATTERN.md, as CLAUDE.md requires for an invariant change.

### [MEDIUM] A trace value as arm has no bound

**Location:** `_lyx/discussion/decision-record.md`, D3, the bullet on a trace value outside the three.
**Issue:** The rule makes log content an arm name, which then appears in every report table and in the samples section.
Today `internal/websterengine/beginbatch.go` logs only `file`, `inline` or `none`, so the edge is reached only by a future layout, which is the case it exists for.
D3 does not say what such a value may hold: a future layout named `raf`, or a value of `unknown`, would merge its forks with the prompt-classified arm of that name, and a value with `|` or a newline would break the markdown tables.
D1 lists no per-arm file, so file names are not reached as long as intermediates stay per-fork; the fix's charset bound covers that case too.

### [MEDIUM] Batch resolution can pick another run's batch

**Location:** `_lyx/discussion/decision-record.md`, D7, the Batch resolution bullet.
**Issue:** `resolve-batches.sh` runs `git log --all -S"$ts"` over the whole records store, and D7 ports it unchanged.
The store holds every pair's records branch and 262 `archive/*` tags (checked in `loomyard-weft`), and the hub runs several pairs at once (four worktrees live now), so a batch of another run spawned in the same six-second window is a real candidate.
The first hit wins, and the fork then gets another run's cards and `startSha`; the pack check reports plausible but wrong classes rather than `notree`.
Restricting the search to the fork's own run's refs removes the collision and keeps the heuristic only for timing.

### [LOW] `--runs` matching and character set

**Location:** `_lyx/discussion/decision-record.md`, D2, the `--runs` bullet and its exit-2 rule.
**Issue:** "Project directory belongs to those worktrees" leaves exact versus prefix match open, and the project directories (`ls ~/.claude/projects`) show that slugs such as `run-pane-timing` contain other slugs as prefixes.
Claude Code encodes every non-alphanumeric character as `-`, so a run name with `_` or `.` can never equal a directory suffix, and the allowed set admits `.` and `..`, which D3's trace join turns into `<hub>/../.lyx/logs`.
The rationale "keeps names out of `find` patterns" holds; the gap is the match rule and the dot names.

### [LOW] `--code` required under `--plan`

**Location:** `_lyx/discussion/decision-record.md`, D7, the missing-directory bullet.
**Issue:** D6 reads plan cards from the records repo and the plan writer from transcripts, and `plan-refs.sh` classifies a reference by its text alone (`plan:` prefix, `#`, trailing `/`, extension).
Nothing under `--plan` reads `--code`, so exiting 1 on its absence refuses a run that would succeed.

### [NIT] Sort order has a tie

**Location:** `_lyx/discussion/decision-record.md`, D5, the paragraph on row order.
**Issue:** Arm, project and start time do not form a total order when two forks of one run start at the same timestamp, and the determinism criterion then depends on filesystem order.

## Rejected

- D7's default-path premise, settled as true: pair weft worktrees are worktrees of `loomyard-weft` (`glyph-audit-tool-weft/.git` points into it), the warp pairs share `loomyard/.git`, `git log --all` in `loomyard-weft` sees every pair's records branch and the `archive/<slug>/*` tags, and begin-batch commits `state.json` (`run-pane-timing-weft` log), so the `-weft` sibling and the repo root resolve every pair's records and `startSha`.
- D2, D3 and D7's availability rules agree with each other and with the criteria: batch resolution runs on a trace file alone, a missing records repo then only disables the cross-check, and the exit-1 rule applies only under `--pack-check` or `--plan`; the one disagreement is F6.
- Acceptance criteria coverage: D2's run-name rule falls under the invalid-inputs criterion, D5's per-cell sample size and D6's missing-plan-writer line each have a criterion; the gaps found are the ones named in F1, F3 and F4.
- The test plan against `PATTERN-test-isolation`, `PATTERN-test-speed` and `PATTERN-lyx-temp-root`: `t.TempDir()` is allowed and lands in the run directory under `tmuxkit.Main`, and the `TestMain` shape matches `tools/tokencount/testmain_test.go`.
- Git use against `PATTERN-fabric-git`: its background states code outside the `lyx` binary is not bound, and the tool's verbs are read-only.
