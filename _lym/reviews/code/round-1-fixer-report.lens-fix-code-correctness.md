```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness lens over the fixer's change set (F1 through F9).

**Findings**

1. **LOW** — `unreadScanSkippedDirectories` in `instructionfiles.go`.
   - The skip applies to any directory named `vendor` or `node_modules` at any depth, with no check against `workDir`.
   - Concrete input: a session anchored at `<root>/vendor/tool`, with `<root>/vendor/tool/AGENTS.md` and `<root>/vendor/tool/CLAUDE.md`.
   - The walk skips `<root>/vendor`, so that `CLAUDE.md` is never named. Before the fix it was named, because it sits beside an `AGENTS.md`.
   - This is a narrow loss of pointers for a session whose own anchor is inside a skipped directory. A guard such as "do not skip a directory that contains `workDir`" would remove it.
   - The added test row only covers a skipped directory below the anchor, so it does not catch this.

**Checked, no finding**

- **F1, `Prepare` now passes `WorkDir: workDir`:**
  - The new row `AnchorThroughALinkGivesTheResolvedWorkDir` reaches the worktree through a symlinked anchor and expects the resolved path.
  - With the old `c.dirs.Anchor` the request's `WorkDir` would be the link path, so the row would fail.
  - The `claudeengine` callers (`skillListing`, hook `Dir`) take a more correct value than before.
- **F4, reordered exemption in `claudeFileIsUnread`:**
  - A heddle-marked override at `workDir`, with an `AGENTS.md` beside it, still returns false, as before.
  - A marked override with no `AGENTS.md` and a `CLAUDE.md` with imports now returns false; before it fell to the import branch and returned true.
  - The new row would fail on the old code, because the file would be named.
  - An unmarked override with no `AGENTS.md` still reaches the import branch, as before.
- **F5, dependency-directory skip:** the row with `src/app/node_modules/dep/CLAUDE.md` would be named on the old code, since it lies below `workDir`, so it catches the defect.
- **F6, `projectDocMaxBytes` refusal row:**
  - A self-link makes `os.Stat` return ELOOP, which is not `ErrNotExist`, so the refusal path is reached.
  - The row asserts the "examine the instruction file" text, a way forward and no `prompt.md`.
  - It does not depend on running as root.
- **F6, hook wiring test:**
  - Without `RunHook` in the registry, the hook would be listed as omitted into warnings rather than the launch line, so the marker assertion fails.
  - The test is tier 1 and non-Windows.
- **F6, heddle render test:** with `Root` empty, `filepath.Rel("", abs)` errors, so `recordImports` writes an empty path list and the `docs/imported.md` assertion fails.
- **F3, `TestUserSetup_UnresolvableConfigDir`:** with `CLAUDE_CONFIG_DIR`, `HOME` and `USERPROFILE` empty, `claudeConfigDir` errors. This gives one warning naming the variable and a degrade path with no global file and no plugin items, which the assertions match. The test is not parallel because it uses `t.Setenv`.
- **Claude-engine rows:**
  - The no-`installPath` row works: `writePluginInstalls` writes `installPath: ""`, which hits the warning-and-break path.
  - The unreadable-plugins-file row works: a directory at that path fails to read for every user.
  - The unreadable-`SKILL.md` row works: a directory at `SKILL.md` makes `ReadFile` fail while its parent is a directory, so it warns.
  - The invalid-matcher row (`[`) reaches the "no pattern" omission.
  - The `type: http` row yields no run and no omission.
- **F7:** `append([]string{tt.wantEffort}, tt.skills...)` builds a fresh slice, and it is only reached when `wantEffort` is non-empty.
- **F8, F9:** the keep markers use the existing `//` blank line then `//testtiming:keep` form. The restored help text matches the code in `renderEveryWorktree`, which still walks every live worktree and still warns on a failed listing.
- **Gate:** tiers 0 and 1 passed on the four touched packages, and every new test showed PASS in the logs.

**Tool calls:** none, because my context already held every changed file and the gate output.
