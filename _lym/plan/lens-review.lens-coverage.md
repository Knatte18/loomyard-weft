```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked every decision, acceptance criterion and branch the cards introduce against a planned test row.

**Findings**

1. **MEDIUM, D6 / AC "thread cap from codex.yaml".** No row reads `max_subagent_threads` from codex.yaml. Every `TestPrepare_SubagentTerms` row asserts `=16`, which is the template default, so a hard-coded 16 would pass. Evidence: card 6, Tests, "Every row asserts `agents.max_concurrent_threads_per_session=16`". Fix: add a row that sets e.g. `max_subagent_threads: 24` in the local layer.

2. **MEDIUM, D5 order (pointers placement).** No row checks where the instruction-file pointers sit relative to the user's instruction files and the skill listing.
   - Card 10's test has no user-setup source, so neither neighbour exists, and its assertion reads "after the creel head and before no skill listing".
   - Card 9's order row has no pointers.
   - Evidence: card 10 Tests; card 9 `TestPrepare_UserSetup`.
   - Fix: a card-10 row with a fake source (files plus listing) and an unread CLAUDE.md, asserting files → pointers → listing.

3. **MEDIUM, D8 AC "a failing hook is left out with a warning".** The warning goes only to `logger.Warn` and no row asserts it; the rows assert `Omitted` alone.
   - Card 9 never says what `Prepare` does with `UserSetup.Omitted`, so it is dropped.
   - The record's "marked omitted (D1)" and the Exports line ("a runner that lists each hook unrun") therefore have no observable channel through `Prepare`.
   - Evidence: card 8, "logged at Warn … appended to `Omitted`"; card 9 Intent has no use of `Omitted`.
   - Fix: route the warning through the engine's injectable `c.warn`, assert it, and state where `Omitted` surfaces.

4. **MEDIUM, card 8 branch.** Two branches have no row:
   - an unreadable or malformed `hooks/hooks.json` ("a warning and an omitted entry naming the file");
   - a hook whose output is empty, or JSON with no `additionalContext` (the item should be skipped).

   Evidence: card 8, Intent vs Tests.

5. **MEDIUM, card 7 branch.** The error paths of `installedPluginRoots` and `skillListing` have no row and no stated outcome.
   - Card 9 makes any source error refuse every Codex launch.
   - Claude's own `sessionScopeFor` only logs a malformed `installed_plugins.json` and goes on (`internal/shuttleengine/claudeengine/scope.go`, `sessionScopeFor`).
   - So a malformed plugins file would block all Codex launches but no Claude launch, and nothing tests this.
   - Also untested: a SKILL.md with no `description:`, and a plugin whose `installPath` is missing.

6. **MEDIUM, D8 fidelity (skill listing).** The listing covers plugin skills and the config directory's `skills/`. It leaves out project skills under the worktree's `.claude/skills`, which a Claude session also lists. The record says "every skill Claude's session scope lists". Evidence: card 7, "each skill under `skills/*/SKILL.md` of a plugin … and each user skill under the config directory's `skills/*/SKILL.md`".

7. **MEDIUM, card 4 branches.** These record branches have no row:
   - an import resolving outside the worktree is left out (the record's own bound);
   - an empty `Root` writes no record;
   - a failed record write becomes a render warning;
   - an unreadable record gives none plus a Warn;
   - the record is rewritten empty when the target holds no text, so a dropped import stops matching.

   Evidence: card 4 Intent vs its two test bullets.

8. **MEDIUM, D9 fidelity (pointers).** The record names each CLAUDE.md "one below the working directory or beside an `AGENTS.md`". Card 10 narrows the second case to directories on the path from root to working directory. A CLAUDE.md beside an AGENTS.md in a sibling directory, off the path, is never named. Evidence: card 10, "one in a directory from the root to the working directory that also holds an `AGENTS.md`".

9. **MEDIUM, card 10 refusal.** "A walk or stat error other than a missing file refuses the launch" has no named reaching input and no row. Rubric: "Reachability of a prescribed refusal".

10. **MEDIUM, card 3 branches.** No row reaches:
    - `NotifyWorktreeMadeAt` with a path that does not resolve (warning, walk goes on);
    - the no-seam-installed case (returns "");
    - a failed render that must not fail the walk;
    - the single-pair `Options.Pair` call rendering its pair.

    The only test is the happy-path stale walk. Evidence: card 3 Intent, "A failed render is the seam's logged warning and never fails or skips the walk".

11. **MEDIUM, Constraints fidelity.** Several constraints have no carrying card, and the plan does not say they belong to sibling parts:
    - `PATTERN-heddle-instructions`: "`HEDDLE.md` states the new target". Card 2 changes nothing in `overview.go`, so `HEDDLE.md` names `agents-prompt` only once some entry uses it, and no shipped entry does.
    - The constraints tied to D1, D3 and D12: the read-only lookup package, the stencil edit-directive background, the tolerant manifest record, shuttle's cobra verb, and the tier0 parity test.

    Evidence: decision record `## Constraints`; overview `## Shared Decisions` has no such statement.

12. **LOW, D7 "the Claude orch included".** No row runs `Passages(claude, "orch")` with an `agents-prompt` entry. No row checks that an entry aimed at both `agents-prompt` and `orch-prompt` gives its text once. Evidence: card 2 Tests.

13. **LOW, D8 `@` imports.** With the anchor in a subdirectory, Codex reads a root-level CLAUDE.md itself (fallback) with its imports unexpanded. Files named by pointers are also read unexpanded. The record says imports are expanded "in every Claude instruction file a Codex session gets". No card states or bounds this. Evidence: card 4 expands only the anchor's CLAUDE.md; card 10.

14. **LOW, D5 cap boundary.** No row sits exactly at or one byte under `maxArgumentBytes`, so an off-by-one in "passes the cap" goes unseen. Evidence: card 5 `OverTheArgumentCap` row only.

15. **LOW, card 9 seams.** These are unreached by any row:
    - "a nil `SandboxProbe` keeps `runSandboxProbe`": the wiring test uses `danger-full-access`;
    - an unparseable `plugin_hook_timeout` already refused by `LoadConfig` (fine as is);
    - the Exports line "names none of D2's closed terms": no card identifies D2's terms or checks them.

Tool calls made:
- `grep` of `internal/shuttleengine/rundir.go`, `hubreconcile` test helpers and `agentconfig.Load`, to confirm that the run directory's base name is the run id (it is, `createRunDir`) and that `newStaleHub`, `geometryOf` and `agentconfig.Load` exist. My context lacked them.
