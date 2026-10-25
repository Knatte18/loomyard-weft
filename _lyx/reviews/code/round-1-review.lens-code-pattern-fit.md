```yaml
lens: code-pattern-fit
outcome: reported
```

Task: the code-pattern-fit lens, checking each rule against the diff `1c344094a..HEAD`.

**Rule list (held or broken)**
- **`PATTERN-cli-cobra` — held.**
  - `cmd/lyx/main.go` drops the `StartAliasCommand` registration and "start" from the "Available modules" list.
  - The root keeps one cobra subtree per module.
  - `clitree_test.go`, `shortcuts_test.go` and `sandbox_coverage_test.go` are edited in the stat. I did not open them, so the help-tree and registration guards being green rests on the stat alone.
- **`PATTERN-refusal-way-forward` — held.**
  - Every refusal that named `lyx resume` or `lyx pause` now names `lyx tender resume` or `lyx tender pause` with the same arguments. These are in `adoptcli`, `fabricengine`, `hibernateengine`, `loomcli/arm.go`, `pauserecord`, `tendercli` and `tenderengine`.
  - Each message changed in the same commit as the test files that assert it (`adopt_test.go`, `pausehold_test.go`, `hold_test.go`, `record_test.go`, `resume_test.go`, `watch_test.go` and others).
  - The new way forward exists as a real verb. `lyx loom start` replaces `lyx start` in README.
- **`PATTERN-shell-guard-table` / agent-denied steps — held.** No tender verb is newly named to agents beyond what the decision record's auto-mode assumption already states, and the guard table is untouched.
- **CLAUDE.md "Docs land in the same commit" — held.**
  - `internal/shortcutcli/doc.go`, `internal/loomcli/doc.go` (the second-mount sentence is removed), `internal/tendercli/doc.go`, `internal/tenderengine/doc.go`, `internal/adoptcli/doc.go`, `internal/hibernateengine/doc.go` and `internal/battenshed/doc.go` are all updated.
  - `docs/overview.md` loses the convenience-alias line and updates the module table, shortcuts, tender and loom entries.
  - `docs/skills.md`, `README.md` and `tools/sandbox/SANDBOX-CORE-SUITE.md` are updated.
  - A repo-wide grep for `lyx (attach|pause|resume|start)\b` returns nothing outside `.git`, `_lyx` and `.lyx`.
- **CLAUDE.md "Markdown: semantic line breaks" — held.** The edits in `README.md`, `docs/overview.md`, `docs/skills.md` and `SANDBOX-CORE-SUITE.md` keep the existing one-line-per-sentence, one-line-cell structure and add no hard wraps.
- **Comment conventions (scribe:golang-comments, code-quality) — held.**
  - The `newLoomCLI` comment no longer names the removed constructor and says "no constructor can forget one".
  - The `refuseCovered` doc comments are updated in `battencli/arm.go` and `loomcli/pausehold.go`.
  - The removed `StartAliasCommand` and its comments are gone entirely, so no commented-out code is left.
  - Line breaks are not judged, per the rubric.
- **Declared PATTERN changes landed — not applicable.** The directive's constraints section declares no changes, and the diff touches neither `PATTERN.md` nor `pattern/`.
- **Commit messages and departures — held.**
  - Neither commit carries a `Pattern-Departure:` trailer, and none is needed because no entry forbids the answer key's way.
  - The first commit's message matches its diff: a and p and r and prime stay, the root start alias goes, and `lyx loom start` stays.
  - The second commit is a comment re-break only.
- **`PATTERN-test-economy` — held.** The `cli_test.go` change in `internal/loomcli` is deletion-heavy (44 lines, net negative) and was not inspected here.
- **CLAUDE.md "Production" and "Worktrees" rules — held.** The diff has no deploy or install path and no git operations beyond the two commits.

**Findings:** none.

**Tool calls:** none. Everything came from the inherited diff, the PATTERN text and the decision record.
