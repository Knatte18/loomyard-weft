```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out as proposed (F1–F6, F8, F9), looking for a case where the proposal is wrong or breaks something. F7 was fixed with the fixer's own change, so it is out of this lens.

**Findings**

1. **LOW** — `docs/overview.md:423`, the new sentence "installs every shell guard row of the `shellguard` table as a `PreToolUse` hook".
   - Summary: the sentence overstates what `installedGuards` installs.
   - Evidence: `switchInstalled` in `guards.go` gates the python and publish rows on `cfg.DenyPython` and `spec.DenyPublish`. The refuse-raw row exists only for a non-empty `refuse_raw` list. `TestInstalledGuards_CoverEveryRow` has `SwitchesOff` and `NoRefuseList` cases that install fewer rows.
   - The same paragraph already calls the python deny switchable. Following the reviewer's wording makes the overview contradict itself and the code.

2. **LOW** — `guards_test.go` `TestGuardEntries_RenderEveryRowWithItsCodexText` (F3, drop of the `hookEntry(toolMatcherBash, guardCommand(row))` comparison).
   - Summary: nothing now ties `entries[i]` to row i's command.
   - Evidence: the remaining assertions check only the literal prefix `{matcher="Bash",hooks=[` of each entry and `guardCommand(row)` on its own. `guards_run_test.go` runs `guardCommand` directly, and the new `TestPrepare_InstallsGuardsFromItsInputs` builds its expectation through `hookSettings`, which calls `guardEntries`.
   - Case: a `guardEntries` that used the wrong row's command, or the same command for every entry, passes every test. The dropped comparison would have caught it.
   - Fix: assert `strings.Contains(entries[i], tomlString(guardCommand(row)))`.

3. **LOW** — `skillload.go` `isSkillLoadMessage` and `skillLoadNone` (F8, recognising the load message by its fixed text).
   - Summary: a launch whose load line named no file now classifies as verified, where it used to be unverified.
   - Evidence: at launch no root held any skill, so `skillLoadNone` was sent. Later a root gains a skill, so `skillPath` resolves it.
   - Result: `ClassifySkillLoad` finds the none-line, reports `Verified: true`, and lists the skill as Missing, though it was never asked for.
   - The reviewer's fix covered only the paths-changed direction.

4. **LOW** — `skillload.go` `compareVersionFields` (F6, any number above any text).
   - Summary: a hash-named version folder made only of digits outranks a real version.
   - Evidence: a hex or SHA folder such as `123456789012` has no dot, so it parses as a number in field 0 and beats `1.10.0` (field 0 is `1`).
   - The rule is total, but the choice is arbitrary for such names. The change adds no test row for it.

F2, F4, F5 and F9 raised no case against the proposals. The new rows distinguish the swapped inputs: the python-only and publish-only expectations would differ under a swap.

**Tool calls:** none. My context already held every changed file.
