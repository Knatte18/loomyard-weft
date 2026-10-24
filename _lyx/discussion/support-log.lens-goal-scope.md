```yaml
lens: goal-scope
outcome: reported
```

Task: review the decision record against its board entry for goal, scope, missing items, unlicensed additions and hidden deferrals.

**MEDIUM: Decision 9 / Scope. The record keeps three whole-solution gates, where the board asks for one at the final gate.**
- Evidence: the board's Want says "run the whole solution once, at the final gate only". Wanted 4 says "keep the full suite to the gate", singular.
- Decision 9 and Decision 2's bound keep `webster`, `merge-in` and `finalize` at scope `all`.
- The green record does not save the Merge-In run, because Merge-In changes the tree by merging in the parent. So a .NET hub runs the full suite at least twice per task, and three times when Code-Review fixes land.
- The record never says that it departs from the Want, or why. It also gives `docs/dotnet-hub.md` no recommended gate-key scopes for a .NET hub. For example, it could set `webster` to `card` or `diff` and leave `finalize` as the single whole run.
- Fix: decide the .NET hub's gate keys against the "once" want, or state the departure with its rationale.

**MEDIUM: Decision 6 / Scope. "Restore once per run, then `--no-restore`" is replaced by restore once per gate call, and the board licensed no such change.**
- Evidence: the board Want reads "restore once per run, then build and test with `--no-restore` and incrementally". Decision 6 has the build command restore on every gate call, and Scope lists "a lyx-managed restore step" as out of scope.
- The rationale (NuGet's no-op restore) is plausible. But it is an auto-pick that contradicts a stated want.
- The support log lists the decision under rejected alternatives. The Goal and Scope never say that a board want is being reinterpreted.
- Fix: state plainly in Decisions that the want is met by an effectively no-op restore, and why that counts as meeting it. Or build the once-per-run restore.

**LOW: Scope. The before/after measurement, which the board lists as Open, is moved to the operator after landing.**
- Evidence: the board says "Measure on the work machine before and after". Scope moves it out because Models never leaves OSL-1046.
- That is defensible, but the "before" figure has to be taken before this lands, and nothing records how the operator captures a baseline.
- Fix: one line in `docs/dotnet-hub.md` or Scope: take a pre-landing `lyx gate report` baseline on OSL-1046.

**LOW: Decision 7. "Keep build processes warm" is met by doing nothing.**
- Evidence: Decision 7 changes no code, and nothing pins the behaviour.
- That is acceptable. But no acceptance criterion pins that no lyx environment (tier, build, `HolderEnv`) sets `MSBUILDDISABLENODEREUSE` or `UseSharedCompilation=false`. A later change could silently defeat the want.

**LOW: Decision 5. `inert_paths` reaches beyond the board's ask, and its path base is unclear for Models.**
- Evidence: Wanted 4 asks only for doc-only diffs to skip test tiers. The record adds an inert-only carry at whole gates under every scoper, `go` included. This is generic and empty by default, so it is licensed, if broad.
- Decision 5 matches patterns against the worktree-relative path. Models' project root is `src/csharp/NORCE.Models/`, not the git root. The record does not say whether a hub writes patterns relative to the git root or to the tier `dir`, and with a slash the two differ.

**LOW: Decision 10/11. Public docs name a repository other entries call proprietary.**
- Evidence: the `csharp-sandbox-hub` note calls Models proprietary and says it is never cloned outside OSL-1046.
- `docs/dotnet-hub.md` would carry Models' project names, paths and repo rules as its worked example. Issue #609 already names Models, so this is likely acceptable. The record does not say whether the operator wants Models-specific detail in loomyard's docs.

**Checked and holding:**
- The test map, `--filter`, the build step (Wanted 6, including the gate properties passed to the build) and the timing exclusivity (Wanted 5, with correctness runs left unserialised) are covered.
- Doc-only skipping is covered.
- The repo rules arrive as a lens, which fits the board's "not tool features".
- Required 1 and 3 are excluded, as is the baseline.
- No MVP phase, no deferral of design work, and no edit to the task's own board entry.
- The task is the right thing to build: the operator says scoping pays even without XDR slowing .NET.

Further tool calls: none.
