```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each change a card makes, the outside readers I found and the cards that cover them.

**Per change: the set, and the cards that cover it**

1. **`commitBoardWriteAndPush` gains `beforePush`** (card 2). Callers are `holdwrite.go:70` (`WriteHold`) and `registrysetter.go:156` (`RegisterMachine`). Card 2 covers both.
2. **`fixtries.Write` removed, try count moved into the hold** (cards 8 and 9).
   - Production uses: `fixloop.go:280` and `green.go:36` (card 8), and `deps.go:130` `ResetTries` (card 9).
   - Test uses: `green_test.go:150-161` (card 8, `TestGreenEnd`), `fixloop_test.go:116-128` (card 8, `fixFixture.tries`/`setTries`), `check_test.go:496` inside `TestRunCheck` (card 8), `dobbyengine/upgrade_test.go:76` (card 9) and `fixtries_test.go` (card 9).
   - Covered.
3. **`internal/adoptrecord` removed** (cards 5, 6, 9, 10).
   - Production importers: `beamcli/cli.go:408-412` and `adoptsection.go` (card 6), `orchcli/watch.go` (card 5), `beamcli/ownerarchive.go:75` and `dobbydeps.go:263,294` through `UnderAdoptWatch` (card 5), adoptcli (card 9).
   - Test importers: `adoptsection_test.go` and `status_integration_test.go` (card 6), `dobbydeps_integration_test.go:399` and `ownerarchive_integration_test.go:89,99` (card 5), `orchcli/underadopt_integration_test.go` (card 5, deletes both functions).
   - Covered.
4. **`AdoptAs` re-signed** (card 9). Its callers are in `adopt_integration_test.go` at lines 114, 184, 195 and 203, all inside `TestAdoptIntegration_SecondCloneTakesOver`, which card 9 edits. Covered.
5. **`hubCheckDeps` and `failedEnd` re-signed** (card 8). Callers are `checkCmd`, `check_integration_test.go:25` and `TestFailedEnd`. Card 8 covers all three.
6. **`dobbyProgram` takes its value from `orchcli.DobbyProgram`** (card 8). The uses are `beamcli/dobbydeps.go:61,215,266,312,508`. Only the constant's value source changes, so the uses need no edit. Covered.
7. **`lym beam status` adopt section changes shape** (card 6). Readers are `adoptsection_test.go` and `status_integration_test.go:162-192` (card 6). Nothing in contracts or plugins reads the section; `contracts/orchrole/orch-role.md:384` names `lym beam status` but not the adopt section. Covered.
8. **New shell-guard row** (card 4).
   - Claude engine: the settings tests and `settings-golden.json` (card 4).
   - Codex engine: `guards_test.go` builds its expected rows from the table, so it needs no edit.
   - `cmd/lym` no-denied-recovery scan: reads `Forms`, and its scanned surface (specs, stencils, skills, bound modules' strings) holds no `lym adopt --take-hold`.
   - Covered.
9. **Retired `--keep-local`, `--drop-local`, `--discard` and `adopt worker` strings.**
   - `fabriccli/anchor.go:269` and `anchor_test.go:321` (card 11).
   - `landingshed/finalize_owner.go:187-198` and its test (card 11).
   - `fabricengine/anchorstop.go:125,143` and `droplocal_integration_test.go:227,234` (card 3).
   - `beamcli/adoptsection*` and `status_integration_test.go:191` (card 6).
   - `docs/overview.md:788-806` (card 9).
   - `lymdirs/dirs.go:41-42` (card 11).
   - `runproc/doc.go:62` (card 11). **`runproc/end.go:88` is missed** (finding 2).
10. **Doc text naming the adopt record, the adopt pause or the worker.**
    - Covered: `beamcli/doc.go:348-350` (card 6), `orchcli/doc.go:111` (card 5), `fabricengine/junctionnames.go:180` and `fabricengine/doc.go:1205` (card 3), `PATTERN.md:33`, `pattern/PATTERN-dobby-runs.md:3,19-21` and `pattern/PATTERN-daemon-wakeups.md:89` (card 9), `dobbyengine/doc.go` (card 5 edits the file).
    - **Missed:** `runproc/end.go:88` and `tenderengine/doc.go:61` (finding 2).
11. **Ways forward re-worded to the hold's re-run, `lym adopt &lt;hold's subjects&gt;`** (cards 8 and 9; card 2 case 3 refuses a single-subject re-run under a multi-subject hold).
    - Covered by card 8: `failedEnd`'s inline lines, `fixReadError`, `fixFilingError`, `runCheck`'s reconcile and gate-stopped lines, `resolveSubjectWorktree`.
    - **Missed:** `fixBody`, `fixRemovalError`, `removeFixTask`, `runCheck`'s owner-read and owned-elsewhere lines, and fronttask's `adopt-check` hand step (finding 1).
12. **`lym adopt check` envelope gains `fix_task` and `lines`** (card 8). The only caller is fronttask's `startAction`, which starts the check detached and reaps it (`fronttask/hub.go:111-165`). Nothing reads its exit code or its output; see finding 3.

**Findings**

1. **MEDIUM, design. Location: card 8 and card 9, "every way forward names `lym adopt &lt;hold's subjects&gt;`".** The re-worded set is only partly covered. These still word `lym adopt &lt;subject&gt;`, which card 2 case 3 refuses as another set under a multi-subject hold:
   - `internal/adoptcli#fixBody`: the stop condition the fix task's agent reads (`fixloop.go:190`).
   - `internal/adoptcli#fixRemovalError` (`fixloop.go:351`).
   - `internal/adoptcli#removeFixTask` (`green.go:49,57`).
   - `internal/adoptcli#runCheck`: the owner-read error and the owned-elsewhere refusal (`check.go:156,159`).
   - fronttask's `adopt-check` row hand step, `"lym adopt " + subject` (`internal/fronttask/fronttask.go:35`).

   The impact is softened because card 2's refusal names the right re-run itself, but the cards' own claim is not met.

2. **LOW, consistency. Location: card 11, runproc and tender docs.** Two doc comments still describe the retired adopt worker and adopt pause, and no card lists them:
   - `internal/runproc/end.go:88` lists "the adopt worker" among a hub owner's processes. Card 11 edits only `runproc/doc.go` and `RoleAdopt`.
   - `internal/tenderengine/doc.go:61` describes the adopt pause as a current operator pause.

3. **LOW, consistency. Location: card 8 `ImpactSummary` and the `checkCmd` exit rule.** Card 8 says fronttask's `adopt-check` row "reads the exit code alone" and justifies exiting 0 so the row "does not retry it at every settle". In fact `startAction` starts the check detached with `startDetached`, reaps it, and appends its output to a file (`internal/fronttask/hub.go:111,116-127,163`). Nothing reads the exit code, so the stated reason does not hold. The behaviour is harmless, but the stated blast radius is wrong.

Tool calls:
1. Searched internal, cmd, contracts, plugins, pattern, `PATTERN.md` and `docs/overview.md` for "adopt record", "adopt pause", "adopt worker" and their readers. The reading manifest had no doc-text sweep beyond the retired flags.
2. Searched contracts, plugins and cmd for readers of adopt envelope fields and of the beam-status adopt section. The manifest held no consumer search for those outputs.
3. Read the adopt mentions in `plugins/ly/skills/board/SKILL.md` and `contracts/orchrole/orch-role.md`, and searched internal for `"adopt"`. Call 2 found these files but not their text.
4. Read `internal/fronttask/hub.go:95-170` and searched for the `lym s` wiring. The manifest held fronttask's action table but not how an action is started or whether its exit code is read.
5. Searched the orchcli, beamcli, dobbyengine, orchengine, fixtries and tenderengine docs for stale adopt-record, adopt-pause and worker lines, to check which doc files the cards' edit lists leave out.
