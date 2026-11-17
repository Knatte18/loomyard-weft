```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. Run each decision's mechanism end to end with the data each step actually receives.

**List**
- **D1:** Board write for a task under a project with sub-projects. Render builds the projection beside the README from the same board view, encodes it with Go's default HTML escaping and writes `window.BOARD = …;`. Purpose met.
- **D2:** Fresh Bolt clone after a render. `board.html` is in the render map and committed, so it arrives with the clone. With the same binary, a re-render writes identical bytes and git sees no change. Purpose met.
- **D3:** A local board write whose commit holds `board.json`, `board-data.js` and `board.html`, replayed onto origin.
  - `claimantOf` routes the files to board because `Claims` now names them.
  - `Apply` ignores their content and re-renders through `RenderToDisk`.
  - The owner-set path returns every name in `Render`'s map to `CommitWritten` (`ownerset.go:286-321`).
  - Purpose met.
- **D4:** `board-data.js` missing. The `<script src>` fails, `window.BOARD` is undefined and the page shows the rerender line. Purpose met.
- **D5:** Operator runs `lym board open` from `<worktree>/<anchor>/internal`, which is the record's own criterion case. Purpose **not met**: the board subtree's `PersistentPreRunE` resolves with the strict `lymcwd.Resolve` before any subcommand's `RunE` runs, and aborts on a subdirectory cwd.
- **D6, `board open`:** The task runs from the prime's anchor folder in `tasks.json`; VS Code's default cwd is the anchor. Purpose met, assuming D5's resolution works.
- **D6, `beam start`:**
  - The hub workspace task has cwd = prime anchor and runs `lym beam start root`.
  - `actingLocation` resolves with strict `Resolve`, and the anchor cwd passes.
  - `root` names the prime.
  - A beam file's task runs `lym beam start <beam>` from the beam's anchor.
  - Purpose met.
- **D7:** A project section with Tasks, Notes and Done becomes `<details open>`/`<details>` inside the existing per-project `<details open>`. Purpose met.

**Findings**

1. **MEDIUM — design — D5 (and the D5 subdirectory acceptance criterion).** Under the record's own wording, `lym board open` never reaches `lymcwd.ResolveWithin`, so it refuses when run from a subdirectory of the anchor.
   - D5 says the verb "reads the board directory as the other board verbs do" and is a subcommand of the existing board cobra subtree (Constraints, `PATTERN-cli-cobra`).
   - That subtree's `PersistentPreRunE` runs for every subcommand except `board` and `intake`. It calls `lymcwd.Resolve(cwd)` and aborts on error, before the verb's own `RunE` (`internal/boardcli/cli.go:125-147`).
   - Strict `Resolve` refuses a cwd below the anchor (`pattern/PATTERN-cwd-resolution.md:6-7`).
   - The criterion "run from a subdirectory of a worktree … hands the opener seam the absolute path" therefore fails.
   - The fix the record needs to state: either the prerun resolves with `ResolveWithin` for `open`, or `open` is exempt from the prerun the way `board` and `intake` are and does its own resolution and config load.
   - A side effect of the exemption: the prerun also loads the board config, and `open` would then load it itself.

Tool calls:
- `sed -n 120,175p internal/boardcli/cli.go` plus `grep actingLocation` in `internal/beamcli/cli.go`. My context lacked the board subtree's `PersistentPreRunE` body, which every board subcommand passes through before D5's verb. It also lacked how `lym beam start` resolves its cwd, which the D6 task's working directory has to satisfy.
