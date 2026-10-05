# Card 10 — Consolidate PATTERN's Testing section

**Create:**
- `pattern/PATTERN-test-speed.md`
- `pattern/PATTERN-test-isolation.md`

**Delete:**
- `pattern/PATTERN-test-tier-purity.md`
- `pattern/PATTERN-tmux-test-isolation.md`
- `pattern/PATTERN-hermetic-git-tests.md`

**Edit:**
- `PATTERN.md`
- `pattern/PATTERN-test-economy.md`
- `internal/gitrepo/internal/gitoracle/testmain_test.go`
- `cmd/lyx/hermeticenv_test.go`
- `cmd/lyx/testmain_test.go`
- `cmd/lyx/tierpurity_test.go`
- `cmd/lyx/tmuxisolation_test.go`
- `internal/battencli/testmain_test.go`
- `internal/boardcli/testmain_test.go`
- `internal/boardengine/boardtest/testmain_test.go`
- `internal/boardengine/testmain_test.go`
- `internal/burlercli/testmain_test.go`
- `internal/burlerengine/testmain_test.go`
- `internal/configcli/testmain_test.go`
- `internal/fabriccli/testmain_test.go`
- `internal/fabricengine/testmain_test.go`
- `internal/gitexec/testmain_test.go`
- `internal/gitkit/doc.go`
- `internal/gitrepo/testmain_test.go`
- `internal/hubforge/testmain_test.go`
- `internal/hubgeom/testmain_integration_test.go`
- `internal/idecli/testmain_test.go`
- `internal/ideengine/testmain_test.go`
- `internal/logger/sink_callsite_integration_test.go`
- `internal/loomcli/testmain_test.go`
- `internal/lyxcwd/testmain_test.go`
- `internal/orchcli/testmain_test.go`
- `internal/pairteardown/testmain_test.go`
- `internal/planglyph/testmain_test.go`
- `internal/preflight/testmain_test.go`
- `internal/reedcli/testmain_test.go`
- `internal/shuttlecli/testmain_test.go`
- `internal/shuttleengine/claudeengine/testmain_test.go`
- `internal/standalonegeom/testmain_integration_test.go`
- `internal/stencilcli/testmain_test.go`
- `internal/treadleengine/gate_lingering_test.go`
- `internal/treadleengine/gate_test.go`
- `internal/treadleengine/testmain_test.go`
- `internal/webstercli/testmain_test.go`
- `internal/websterengine/testmain_test.go`

**Intent:**
PATTERN's Testing section ends with five entries (D8): `PATTERN-test-economy`, `PATTERN-test-speed`, `PATTERN-test-isolation`, `PATTERN-testkit` and `PATTERN-hubforge-fixtures`.
This card does the merge on today's rules and tags;
cards 11, 13, 15 and 17 then add the `llmkit` exemption, the tier table, the LLM rule and the socket rule to these entries in the same commits as their code.

- `PATTERN-test-speed` absorbs `PATTERN-test-tier-purity`, `PATTERN-test-time-seam` and `PATTERN-test-parallel`: an untagged test spawns no git, process, tmux or `lyxbin` and sleeps under a second; a production interval, timeout or clock is injectable per instance and a test shortens it; a test calls `t.Parallel` unless it touches process-global state, which a comment names.
  It carries `(test)` and links its new background file, which holds the tier-purity background's content (the spawn definitions, `cmd/lyx/gitkitspawn_test.go`, `cmd/lyx/tmuxkitspawn_test.go`, the sleep rule) plus the time-seam and parallel rules.
- `PATTERN-test-isolation` absorbs `PATTERN-hermetic-git-tests` and `PATTERN-tmux-test-isolation`: a package whose tests spawn git runs them under `gitkit.HermeticGitEnv()`, and a package with a tagged test file runs them through `tmuxkit.Main`.
  It carries `(test)` and links its new background file, which holds both retired backgrounds' content, enforcement files and allowlist rules.
- `PATTERN-test-economy` keeps its entry line; its background points at `scribe:testing` and `scribe:golang-testing` for the general rules and keeps only what is project-specific or stricter here, including that a redundancy finding names the covering test.

Each entry line stays one line within `internal/pattern`'s `MaxEntryLineChars`, and the overview within `MaxOverviewBytes`;
`TestOverview_RealFilePassesCheck` checks both and that every `pattern/` file is named by exactly one entry, so the three retired background files are deleted here (the time-seam and parallel entries had none).

The enforcement tests keep every check and name the new entry: `cmd/lyx/tierpurity_test.go` reports `PATTERN-test-speed`, and `cmd/lyx/tmuxisolation_test.go` and `cmd/lyx/hermeticenv_test.go` report `PATTERN-test-isolation`.
Every other reference to a retired ID in a code comment, test message, stencil, spec or doc is rewritten to the entry that absorbed it; the target list above is every file `grep -rln` finds for the five retired IDs outside `_lyx/` (discussion artifacts are exempt).
A prose invariant name such as "Test Tier Purity Invariant" is not an entry ID and stays.
Run the same grep after the edit; it returns nothing outside `_lyx/`.

**ImpactSummary:** text-only across PATTERN, its background files and comment or message strings in the listed files; no check, allowlist or behavior changes.
