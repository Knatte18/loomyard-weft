```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff's changes force outside the lines it touches.

**The set, per change, covered or not**

1. **`reportcli.primeGeometry` renamed to `PrimeGeometry`.** Callers are `reportcli/{cli,cost,raddle,timing}.go` and `retrocli/cli.go:215`. All are updated, and no reference to the old name is left: the only `primeGeometry` left is `reedengine`'s own unrelated method. Covered.
   - Rule set: `pattern/PATTERN-cli-cobra.md` "Package naming" lists every cli package that reaches past its own engine. `retrocli` now imports `reportcli`, `reportengine`, `boardengine` and `fabricengine`, and that list is not updated. **Not covered (F2).**
2. **New `lym retro` module.**
   - Root registration in `cmd/lym/main.go`: covered. Covered: the "Available modules" list in `Long`.
   - `docs/overview.md`, both the package tree and the module list: covered.
   - Sandbox allowlist in `cmd/lym/sandbox_coverage_test.go`: covered.
   - A search of `README.md` and `docs/` found no other list of the modules that names `lym report`.
3. **New stencil `retro-template-task`.**
   - Registry entry and embed in `contracts/stencils/stencils.go`: covered.
   - Citation guard prefixes (`contracts/`, `docs/`, `internal/`): the stencil names `docs/` paths only through markers, so covered.
   - Parent directive, role skills and color: not applicable, since the stencil is a board body and not a spawned role's stencil.
   - Engine blocks: not engine-specific text for the agent's own engine.
   - Stencil wording against the board entry:
     - routing, with a new prompt sentence as the last resort: covered;
     - most severe first: covered;
     - tracing each proposal to a moment in a named run: covered;
     - the exact read cap: covered;
     - the question set: covered.
   - Every marker the stencil uses is filled: covered.
4. **New `tools/retro` scripts.**
   - `PATTERN-lym-files` single declarer: `retro.sh` names `_lym` inline, outside a variable block. **Not covered (F1).**
   - `PATTERN-fabric-boundary`: code outside the `lym` binary is not bound. Covered.
   - `PATTERN-test-isolation`: `testmain_test.go` calls `HermeticGitEnv` and `tmuxkit.Main`. Covered.
   - `PATTERN-test-speed`: tier1, `!windows` constraint, no tier tag. Covered.
   - `PATTERN-code-conventions` "No hard-coded choice / Engine": the script reads only Claude Code transcripts. **Not covered (F3).**
5. **New `retroengine` package.** It does not import `lymcwd`, and its paths and names are named constants. Covered.
6. **Change description (`_lym/landing/summary.md`).**
   - The `tools/retro` test claim is not borne out. **Not covered (F4).**
   - The other claims hold against the diff.
7. **Commit messages and departures.** There are no `Pattern-Departure:` trailers in the range, so there is nothing to judge.
8. **Declared PATTERN changes.** The directive declares none, so there is nothing to check.

**Findings**

- **F1. BLOCKING, design.** `tools/retro/retro.sh:62,66,69,79` (the lines in the diff that hold `_lym/usage`, `_lym/shed/$slug/status.json` and `_lym/reviews`).
  - `retro.sh` names `_lym` inline in its git commands. `PATTERN-lym-files` allows only `internal/lymdirs` to name `_lym`, with one exception: "a `tools/` script's variable block".
  - Evidence: `pattern/PATTERN-lym-files.md`, the "Durable and ephemeral state" rule line and "Single declarer". `tools/forkaudit/forkaudit.sh` follows the rule, using `$STATE_PATH` and `$PLAN_DIR` at lines 344 and 411.
  - Fix: declare the three record paths once in a variable block at the top of `retro.sh` and use the variables.
- **F2. MEDIUM, consistency.** `pattern/PATTERN-cli-cobra.md`, "Package naming" deviations.
  - `retrocli` calls `reportcli` (another cli package), `reportengine`, `boardengine` and `fabricengine` beyond its own `retroengine`. The deviation list records each such cross-module cli dependency, for example `shedcli` using `loomcli` and `battencli`, or `beamcli` calling `fabricengine` and `boardengine`. The diff adds no line for `retrocli`.
  - Evidence: `internal/retrocli/cli.go:21-31` and `:215`; `pattern/PATTERN-cli-cobra.md` lines 75-88 of the background.
  - Fix: either add a `retrocli` deviation line, or move `PrimeGeometry` out of a cli package so that `retrocli` does not import `reportcli`.
- **F3. LOW, design.** `tools/retro/retro.sh`, header and transcript loop, and `contracts/stencils/retro/retro-template-task.md` ("The reading" section).
  - The figures read only Claude Code transcripts (`~/.claude/projects`, `&lt;session&gt;/subagents/`). A run on another engine shows up only as "no transcript directory".
  - `PATTERN-code-conventions` says "no code path assumes one ... agent engine".
  - Evidence: `pattern/PATTERN-code-conventions.md`, "No hard-coded choice / Engine"; the `PROJECTS` default in `retro.sh`.
- **F4. LOW, consistency.** `_lym/landing/summary.md`, the line in the Tests section about `tools/retro`.
  - It claims the test checks "every table it writes". `TestRetroScript_WritesTheFiguresOfEachRun` asserts `sessions`, `refusals`, `rows`, `keys` and `missing`, plus four `figures.md` lines. It never reads `facts.jsonl` or `usage.jsonl`.
  - Evidence: `tools/retro/retro_integration_test.go`, the `tables` slice and the `sessions` decode.

**Tool calls**

1. Grep across `docs/`, `README.md` and `contracts/` for other lists of modules, stencils or tools. The context had no search beyond `docs/overview.md`.
2. Grep of `docs/overview.md` for its tools tree, of enforcement tests for a rule against one cli importing another, and of every importer of `reportcli`. The context lacked the callers outside `reportcli` and any such enforcement test.
