---
verdict: APPROVED
---

## Scope against the board entry

The board entry `creel-sandbox-suite` asks for a new `tools/sandbox/SANDBOX-CREEL-SUITE.md` holding S9 and C1 to C13 with their ids kept, carrying the black-box rule, the controlled tmux and creel exceptions, pre-conditions and the session-log format creel needs.
It also asks that the shuttle suite keep only shuttle scenarios, and that `tools/sandbox/suite.go`, its tests and the `PATTERN-sandbox-coverage` guard follow the move.

- D1 builds the creel suite file with the sections the entry names, and the scenarios moved as written.
- D2 keeps every id, S9 included.
- D3 cuts the shuttle suite down to S1 to S8, with no creel scenario, fixtures or exceptions left.
- D4 adds the `creel-suite` subcommand in `suite.go` and `main.go`, and its tests are an acceptance criterion.
- D5 leaves the coverage guard unchanged, because it globs `tools/sandbox/*SUITE.md` and so follows the move with no edit; the criteria require it to pass with creel and shuttle each covered.

The launchers and the `docs/sandbox-howto.md` lines follow the `<module>-suite` shape every other suite has, which the entry names as the reason for the change, and CLAUDE.md's docs-in-the-same-commit rule.
D3 also lists S4 to S7 in the shuttle session log, which today leaves them out.
That fix is inside a section D3 rewrites anyway, so it is no deviation.

Scope Out leaves the creel stage notes `creel-notices` and `creel-agents` alone, so the design neither duplicates nor contradicts them.
It also leaves the #673 anchor lines to `reed-add-parentless-placement`.

## Overlap with runs in flight

`lyx loom peers` lists no landed parent commits.

- `reed-add-parentless-placement` (row 7, own row 3): this run moves the parentless `lyx reed add` lines in S9, the creel fixtures and C13 unchanged.
  That peer changes `lyx reed add`'s default to `own-window`, removes the #673 workaround from CLAUDE.md and sweeps only the reed suites (`SANDBOX-REED-SUITE.md`, `SANDBOX-REED-WATCH-SUITE.md`).
  The shuttle suite names no `below-parent`, so the moved lines need no edit under that peer's acceptance grep.
  The designs do not conflict: this run makes no anchor decision, and its Scope Out leaves the anchor to the #673 fix.
  Landing order: `reed-add-parentless-placement` is further along (row 7 against 3), so it lands first and this run takes its changes at Merge-In.
- `orch-role-stencil-fixes` and `loom-stencil-wording` (row 5): their decision records touch no sandbox suite, the sandbox tool or creel, so there is no overlap.
