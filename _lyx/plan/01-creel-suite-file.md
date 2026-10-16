# Card 1 — creel-suite-file

**Create:**
- `tools/sandbox/SANDBOX-CREEL-SUITE.md`

**Edit:**
- `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md`

**Intent:** Move creel's sandbox scenarios out of the shuttle suite into their own suite file, keeping every scenario id, and file the board note D6 asks for.

`tools/sandbox/SANDBOX-CREEL-SUITE.md` opens with `# SANDBOX-CREEL-SUITE -- lyx creel black-box agent suite` and the shuttle suite's section set, written for creel (D1):

- `## What this is`: a structured test-loop exercising `lyx creel` against a live tmux server and a logged-in claude in the sandbox Hub's Fabric repo; an agent drives it, an operator watches.
- `## Pre-conditions`: the shuttle suite's four items adapted.
  Item 1 keeps the fresh dev binary and its three in-pane checks (`command -v lyx`/`where lyx`, `LYX_BIN`, `claude`) verbatim.
  Item 2 materializes the hub as the shuttle suite says.
  Item 3 requires tmux, PowerShell 7, which reed's strands need, and a logged-in `claude` on PATH, since S9 adds a `claude` strand; its "note that as the session outcome rather than treating it as a creel defect" line names the `**Covers:** creel` tag on S9 as satisfying `sandbox_coverage_test.go`.
  Item 4 requires a worktree wired by `lyx fabric clone`/`lyx fabric add`, which `lyx reed` needs.
- `## Black-box rule`: the shuttle suite's bold paragraph verbatim, then the discovery line naming `lyx creel`, `lyx reed`, their subcommands and `--help`.
  No controlled tmux exceptions section: the shuttle suite's tmux exceptions, the S2 operator-attach line among them, stay in the shuttle suite.
- `### Controlled creel exceptions`: the shuttle suite's section verbatim, its lead line included, plus one bullet: pane content a scenario judges (S9's typed reminder, C2's single reminder in the scrollback, C6's line not typed again, C7's restored prompt) is observed by the operator through `lyx reed attach` and reported back by the agent, as C12's operator-assisted check is.
- `## Fingerprint header`, `## How to run a scenario`, `## Verdict key`, `## Capturing findings`: the shuttle suite's text with `lyx creel` for `lyx shuttle`; the how-to-run discovery line names `lyx creel` and `lyx reed` with their `--help`; the JSON example's `ref` is `"C2"`, and the `ref` bullet reads `` `ref` is the scenario id (`S9`, `C1`-`C13`). ``
- `## Scenarios`: S9, the `**Creel scenario fixtures.**` paragraph and C1 to C13, copied byte for byte from the shuttle suite in their current order, each with its `**Covers:** creel` line and its `---` separators.
  The parentless `lyx reed add` lines get no `--anchor own-window`, and C13 moves as written.
- `## Session log format`: the shuttle suite's block with one line each for S9 and C1 to C13.
- `## Notes`: creel scenarios go here as `C` scenarios with a `**Covers:** creel` line; shuttle scenarios stay in `SANDBOX-SHUTTLE-SUITE.md`.

`tools/sandbox/SANDBOX-SHUTTLE-SUITE.md` loses S9 with its trailing `---`, the creel fixtures paragraph, C1 to C13 and the `### Controlled creel exceptions` section (D3).
Its `ref` bullet reads `` (`S1`-`S8`) ``, its session log lists one line each for S1 to S8 (adding today's missing S4 to S7 and dropping S9 and the C lines), and its Notes bullet about creel becomes: creel scenarios live in `SANDBOX-CREEL-SUITE.md`.
S8 becomes the last scenario, so the file goes from S8's `**Verdict:**` line straight to `## Session log format`.
Nothing else in the shuttle suite changes, and no scenario's goal, watch text or verdict rule changes in either file.

`cmd/lyx#TestSandboxCoverage_AllModulesCoveredOrExcluded` globs `tools/sandbox/*SUITE.md`, so creel stays covered through the new file's `**Covers:** creel` lines and shuttle through S1 to S8 (D5); it is left as is and runs under the plan's verify.
The suite text itself is checked by review, not by a new test.

After the files are written, file the board note D6 asks for, once, through `lyx board upsert` (an upsert by slug, so a re-run writes the same note again rather than a second one):
`{"slug":"creel-sandbox-c13-stacked-rewrite","kind":"note","labels":["enhancement","creel"],"title":"Rewrite C13's stacked half for window-per-strand","brief":"C13's stacked 3-row half cannot be set up once reed-add-parentless-placement lands"}` with a `--body-file` body saying that `reed-add-parentless-placement` removes the stacked layout and `collapsed_rows`, refuses `below-parent` and `top`, and moves a strand pane left in the shell window into its own window, so C13 in `tools/sandbox/SANDBOX-CREEL-SUITE.md` needs its stacked 3-row half rewritten against the window-per-strand layout while its hand-added half stays as is.
The body file is a scratch file under `.scratch/`, removed by its literal path afterwards.
This writes another board entry, never this task's own (`creel-sandbox-suite`).

**ImpactSummary:** two suite text files and one new board note; `shuttleSandboxSuiteMD` embeds the cut file unchanged in mechanism, and the coverage guard keeps creel and shuttle covered with no code change.
