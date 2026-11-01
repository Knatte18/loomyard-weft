# Card 6 — render-button-row

**Create:**
- `plan:internal/reedengine#buttonSpecOption` -> `const buttonSpecOption = "@lym_button_spec"`
- `plan:internal/reedengine#buttonDirOption` -> `const buttonDirOption = "@lym_button_dir"`
- `plan:internal/reedengine#buttonSpec` -> `type buttonSpec struct`
- `plan:internal/reedengine#buttonSpecEntry` -> `type buttonSpecEntry struct`
- `plan:internal/reedengine#buttonRangePrefix` -> `const buttonRangePrefix = "lymbtn"`
- `plan:internal/reedengine#Engine.pinButtonRowLocked` -> `func (e *Engine) pinButtonRowLocked(target string)`
- `plan:internal/reedengine#Engine.composeButtonRowLocked` -> `func (e *Engine) composeButtonRowLocked() (plain, agent string, spec buttonSpec)`
- `plan:internal/reedengine#TestPinButtonRowLocked` -> `func TestPinButtonRowLocked(t *testing.T)`

**Edit:**
- `internal/reedengine#Engine.pinGeometryOptionsLocked`
- `internal/reedengine/doc.go`
- `internal/reedengine/bar.go`
- `internal/reedengine/follow.go`
- `internal/reedengine#TestPinGeometryOptionsLocked`
- `internal/reedengine#TestAttachArgv_PreflightOnAKnownGoodSession`
- `internal/reedengine#TestUp_BootValidation`

**Uses:**
- `internal/reedengine#placeButtons`
- `internal/reedengine#fillRunPlaceholders`
- `internal/reedengine#fillConfirmPlaceholders`
- `internal/reedengine#ButtonContext`
- `internal/reedengine#ButtonBeam`
- `internal/reedengine#Geometry`
- `internal/reedengine#Config`
- `internal/reedengine#unlitButtonStyle`
- `internal/reedengine#buttonRowPlainOption`
- `internal/reedengine#buttonRowAgentOption`
- `internal/reedengine#resolveShellPath`
- `internal/reedengine#installFakeTmux`
- `internal/reedengine#newTestEngine`
- `internal/shell#Shell`

**Intent:**
Render the session's buttons into session options at the status pins (D5), so line 3 draws them and a click (cards 8 and 9) runs the command drawn under it.

A new file `internal/reedengine/buttonrow.go`:
- `buttonSpec` is the JSON stored in `@lym_button_spec`: `shell`, the absolute path of reed's configured shell (`resolveShellPath(e.cfg.Shell)`); `flag`, `shell.ForGOOS().CommandFlag()`; `dir`, the run directory; and `entries`, one `buttonSpecEntry` per drawn button: `index`, `label`, `run` with the static placeholders filled, `confirm` filled the same way, and `strand`, true for an entry whose `run` or `confirm` still holds `{strand}` after the static fill, which only a `window: agents` entry may (card 3).
- `composeButtonRowLocked` composes the row:
  - A nil `geom.Buttons` (standalone) gives empty rows and a zero spec, so line 3 shows `follow` alone.
  - Otherwise it calls `geom.Buttons()`; an error is logged at Warn naming it, and gives the same empty result (D4).
  - The entries are `placeButtons(e.cfg.Buttons, ctx.Place, "reed.yaml (the hub base or the worktree file)")` then `placeButtons(ctx.MachineButtons, ctx.Place, "desktop.machine.yaml")`, reed's first (D3); card 3 says why reed's label names both files.
  - The static values are `slug`, `worktree` and `session` from the context.
    `ctx.Beam()` is called only when `ctx.Place` is `ButtonPlaceTask` or an entry's `run` or `confirm` holds `{beam}`; an error is logged at Warn and gives the empty result.
    A non-empty `Anchor` is the `beam` value; an empty one leaves out every entry using `{beam}`, with one Warn per entry naming its label, the missing beam's `Name` and the way back: take the beam onto this machine with `lym adopt <beam>`, after which the row re-renders at the session's next attach (D4).
  - The run directory is, for a task, `Anchor`, or `PrimeAnchor` when `Anchor` is empty, and for the prime and a beam `geom.AnchorPath` (D7).
  - An unresolvable shell is logged at Warn naming the `shell` key and gives the empty result, since no click could run.
  - The kept entries are numbered `0..` in row order; entry `N` is drawn `#[range=user|lymbtn<N> ` + `unlitButtonStyle` + `] <label> #[norange list=on default] `, `lymbtn<N>` staying within tmux's 15-byte limit for `range=user|X`.
    The plain row holds the entries without `window: agents`, the agent row every entry, both in config order.
  - Each entry's `run` is `fillRunPlaceholders(shell.ForGOOS(), run, values)` and its `confirm` `fillConfirmPlaceholders(confirm, values)`; `{strand}` stays for the click.
- `pinButtonRowLocked(target)` sets, in one tmux invocation of four `set-option -t <target>` commands joined by `;` argv separators, `buttonRowPlainOption`, `buttonRowAgentOption`, `buttonSpecOption` (the spec's JSON, or the empty string for the empty result) and `buttonDirOption` (the run directory as one word quoted by `shell.ForGOOS().Quote`, which the click line's directory change reads as is (card 9), or empty), so no click sees a row and a spec from different renders.
  A failure is logged at Warn and ignored, like the other status pins.
- `pinGeometryOptionsLocked` calls `pinButtonRowLocked(target)` right after the status option pins, so boot and every attach pre-flight render it, and a config edit or a pair marked a beam after boot reaches a live session at its next attach or boot.

Docs: the two row options stay display only; `@lym_button_spec` and `@lym_button_dir` are decision-bearing beside `@lym_follow`.
`bar.go`'s header sentence "Every option read here is display only but one", `follow.go`'s header sentence calling `@lym_follow` "the one decision-bearing session option reed reads", and the `doc.go` sentences saying the same (the strand options paragraph and the focus-rules paragraph) change to name `@lym_follow`, `@lym_button_spec` and `@lym_button_dir`, and `doc.go` gains a paragraph on the action row: the render points, the composition order, the two rows, the spec option and the run directory.

Tests, `tier0`:
- `TestPinButtonRowLocked`, a new table test in a new internal test file, drives `pinButtonRowLocked` against the fake tmux with a stub `geom.Buttons` and `e.cfg.Shell` set to `sh`, in a file under `//go:build tier0 && !windows` since `sh` resolves only there, and decodes the spec JSON.
  Its rows read Warns through `logcapture.CaptureVerbose`, so it does not call `t.Parallel`, and a comment names the logger's process-global output as the reason.
  Rows:
  nil closure → one invocation setting all four options empty;
  a closure error → the same, and a Warn;
  `e.cfg.Shell` `no-such-shell-lym` with valid entries → the same, and a Warn naming `shell`;
  a task whose `Beam` returns an error → the same, and a Warn;
  a prime with reed's `prime: []` and `beam: [A]` plus machine `beam: [M]` → rows `lymbtn0` A then `lymbtn1` M;
  a `window: agents` entry with `{strand}` → `strand` true in the spec, and a `window: agents` entry without `{strand}` → `strand` false, both in the agent row and not the plain row;
  an invalid entry beside a valid one → only the valid one drawn, as `lymbtn0`;
  a task whose `Beam` returns `Anchor` `/b eam` → spec `dir` `/b eam`, `@lym_button_dir` `sh.Quote("/b eam")` and a `{beam}` entry's run holding `sh.Quote("/b eam")`;
  a task whose `Beam` returns an empty `Anchor` → its `{beam}` entries left out with a Warn naming the beam and `lym adopt`, `dir` the `PrimeAnchor`;
  a prime whose entries use no `{beam}` → `Beam` never called (the stub counts calls), `dir` the geometry's anchor;
  a `{slug}` value `a#b` in `confirm` → spec confirm holds `a##b`;
  a second render after `e.cfg.Buttons` changes → the second invocation holds the new rows;
  a failing invocation → a Warn, no panic.
  Each row asserts the exact single `set-option ... ; set-option ...` argv for its rows and dir.
- `TestPinGeometryOptionsLocked` gains the row invocation in `wantOptions`, right after the status option pins, and its pin counts grow by one.
- `TestAttachArgv_PreflightOnAKnownGoodSession`: `wantSetOptionCalls` is `14 + wantShellPins`.
- `TestUp_BootValidation` gains the row `ButtonGroupMappingPasses`, whose `configure` sets `cfg.Buttons` to a decoded `buttons: {task: {label: x}}` node, with `notErr` `buttons`: Up's eager boot validation never refuses on a button, so a bad button cannot fail `lym reed up` (D2's acceptance criterion); the render that follows skips it, as `TestPinButtonRowLocked` pins.

**ImpactSummary:** the geometry pin gains one tmux invocation per boot and attach, which reads the geometry's button closure; the tests counting the pin's set-option calls change; no other caller.

**Commit:** 6: Render the configured buttons into the session's status options
