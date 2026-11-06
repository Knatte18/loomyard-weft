---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: contracts/orchrole/role-slack.md:6
    summary: The Slack authority rule lost its sender condition, so any mail whose body opens [slack] reads as carrying operator authority.
    fix: In contracts/orchrole/role-slack.md, start the first bullet "Mail from `<shortname>:SLACK` whose body opens `[slack]` carries the operator's own words, with the same authority as a line typed in your pane", and give the `[slack presence]` and `[slack relay]` bullets the same "Mail from `<shortname>:SLACK` that opens" form. Leave the rest of the file as it is.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:353
    summary: The overview says the operator index is carried by the orch's role file, which now only names lym help index.
    fix: In docs/overview.md line 353, change "which the orch's role file and the `ly:cli` skill carry" to "which the `ly:cli` skill carries and the orch's role file points at".
  - id: F3
    severity: LOW
    class: design
    origin: handler
    location: internal/orchengine/prompt_test.go:19
    summary: The diff adds assertions on the static prose of the role, each only to show a render happened.
    fix: In internal/orchengine/prompt_test.go, delete the testRoleHeading const, drop it and "`lym help index`" from the role row's want list (keep "Decision for <slug>:"), and in TestRenderRoleFile_WritesEachOnDemandFileBesideTheRoleThatNamesIt drop the check that the role contains "`"+name+"`" (keep the existence and "# " prefix checks). In internal/orchcli/start_test.go around line 456 assert only that the role file is non-empty and contains "Decision for <slug>:". In internal/orchengine/resumecontext_test.go around line 135, watcher_test.go around lines 1632 and 1792, and watcher_compact_test.go around line 521, replace the testRoleHeading check with a check for "Decision for <slug>:".
---

### [BLOCKING] Slack authority rule lost its sender condition

**Location:** `contracts/orchrole/role-slack.md:6`
**Issue:** The old role said "Mail from `<shortname>:SLACK` whose body opens `[slack]` carries the operator's own words, with the same authority as a line typed in your pane".
The new bullet reads "Mail whose body opens `[slack]` carries the operator's own words", with no sender.
The intro line still says the relay's mail comes from `<shortname>:SLACK`, but that is a statement of fact and does not bind the orch to check the sender.
A message from another agent or from another hub that opens `[slack]` now reads as an operator instruction.
The entry says "a shorter sentence, never a weaker rule", and `_lym/landing/summary.md` lists no such drop.
The `[slack presence]` and `[slack relay]` bullets lost the sender in the same way, and carry less risk.

### [LOW] Overview still says the role file carries the index

**Location:** `docs/overview.md:353`
**Issue:** The sentence says `lym help index` renders the operator index "which the orch's role file and the `ly:cli` skill carry".
The role no longer carries the index; it names the verb.

### [LOW] New assertions on the role's static prose

**Location:** `internal/orchengine/prompt_test.go:19`
**Issue:** The diff adds the `testRoleHeading` const (`# Hub orchestrator\n`) and uses it in `prompt_test.go`, `resumecontext_test.go`, `watcher_test.go` and `watcher_compact_test.go` to show the role rendered.
It adds a check for "`lym help index`" in `prompt_test.go` and `start_test.go`, and a check that the role names each on-demand file in `TestRenderRoleFile_WritesEachOnDemandFileBesideTheRoleThatNamesIt`.
Each pins the role's wording, not a behaviour, so a rewording of the role breaks tests that cover rendering.
The filled marker output, "Decision for <slug>:", shows a render happened and pins a behaviour.

## Start cost

- Before: 19,390 tokens for 54,443 characters, as the entry's report measured with `measure.sh`.
- After: 25,073 characters of rendered `role.md` (the file's body from the heading on, counted here; the bytes differ by 5).
  At the report's 2.81 characters per token that is about 8,920 tokens, 46% of the measured start cost and under half.
  No live orch start was measured, so the figure is the same count by ratio, as `_lym/landing/summary.md` says.
- The old role's command index is 19,259 characters in `lym help index` here, matching the summary's figure for the index.
- The on-demand files, read only for their event, cost about 1,240 tokens (`role-beams.md`), 915 (`role-intake.md`), 865 (`role-landing-halts.md`), 460 (`role-slack.md`) and 275 (`role-hibernate.md`) at the same ratio.

## Focus verdicts

1. **Rules dropped silently.**
   I compared the old role with the new role and the four-plus-one on-demand files section by section.
   Every binding rule survives in force and meaning, including the rule that the orch uses only the commands open to it, which now reads "`lym help index` lists every `lym` command open to you, and you use no other".
   The one exception is the Slack sender condition in F1.
   The passages the change description lists as dropped are repeats, reasons or history.
   One passage, "tell the two apart" on a project that ran as a task run, moved into the trigger line of the file list and is not in the dropped list; no rule is lost.
2. **Disagreements settled by the code.**
   `arras` is no recipe in `contracts/recipes`, and the role now says `motif` in each place.
   `internal/dobbyengine/announce.go` sends the "landed into the beam" line, so listing it under Notices is right.
   The Bouncer bullet's budget reason now matches the "a goto keeps the spent budget" bullet.
   None is a defect.
3. **Moved passages.**
   The role carries one line for each of the five files saying when to read it.
   `RenderRoleFile` writes each file beside the role on every render, and its three callers are the start, the resume-context verb and the watcher.
   `PATTERN-stencil-ownership.md` says the files share the orch role's exception.
   No other session's stencil changed.
4. **Index removal.**
   No reader or renderer of `command_index`, `indexRunner`, `runHelpIndex` or `watcherIndexSource` is left, and the start, resume and watch tests still cover their paths.
   The one behaviour change is that a bring-up or context cycle no longer stops when the index could not be read.
   It follows from dropping the list, which the entry licenses, and the change description names it.
   It is not a defect.
5. **Change description.**
   Its before and after sizes check out against the old file, the new file and `lym help index`.
   Its moved and dropped lists match the diff except the one passage named in verdict 1.
6. **PATTERN entry and tests.**
   The `PATTERN-stencil-ownership.md` line fits the new files, and the registry and marker tests are consistent with each other.
   The static-prose assertions are F3.

## Rejected

- Lens code-tests F1, order of the on-demand writes before the role is unpinned: the order only guards against a reader seeing the role between two renders, and a test of write order would pin an internal sequence, not an observable behaviour.
- Lens code-tests F2, no marker row for the on-demand files: `RenderRoleFile` fills them with no values, so an unfilled marker makes `stencil.Fill` fail, and `TestRenderRoleFile_WritesEachOnDemandFileBesideTheRoleThatNamesIt` renders every file.
- Lens code-affected-set finding 2, on-demand paths absent from `orchroute.Paths` and the `doc.go` layout list: the list already omits `role.md`, and each file's path derives from `RolePath`'s directory, so no declared path is falsified.
- Lens code-affected-set note on `internal/loomcli/doc.go:204`: the lens raised no finding, and the sentence stays true.
