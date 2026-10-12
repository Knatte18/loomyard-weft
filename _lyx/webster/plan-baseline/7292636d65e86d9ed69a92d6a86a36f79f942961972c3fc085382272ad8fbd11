# Card 5 — Inject refuses the interrupt key; the guarded-interrupt PATTERN entry

**Create:**
- `plan:internal/shuttleengine#TestRunner_Inject_RefusesAnyInterruptKey` -> `func TestRunner_Inject_RefusesAnyInterruptKey(t *testing.T)`
- `pattern/PATTERN-guarded-interrupt.md`

**Edit:**
- `internal/shuttleengine#Runner.Inject`
- `internal/shuttleengine#TestRunner_Inject_HappyPath_PlaysEveryInputInOrder`
- `internal/shuttleengine#TestRunner_Inject_DeadStrand_Refuses`
- `internal/shuttleengine#TestRunner_Inject_UnknownGUID_RefusesBeforeTouchingReed`
- `internal/shuttleengine#TestNewRunner_RefusesUnusableToldPaths`
- `internal/shuttleengine#TestNewDetachedRunner_RefusesUnusableToldPaths`
- `PATTERN.md`
- `pattern/PATTERN-pane-text-injection.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D7: `Runner.Inject` stays the one raw key path that skips the ready-pane check, but it refuses the interrupt key, by key name and as a control character in text, so no caller carries it past the guards card 4 added (bound below), and the rule becomes a PATTERN entry in the same commit.

`Runner.Inject`:

- After the told-path check and the empty-inputs check, and before `FindRun` and `requireLiveStrand`, it refuses an input list in which any input carries the interrupt key, by either of two checks:
  - A `Key` that names a `Key` of any input of `r.engine.InterruptSequence()`, compared case-insensitively, with tmux's `C-[` read as `Escape`, since tmux delivers both as the ESC byte.
    Its message: `shuttle: Inject: the inputs hold the interrupt key %q, which Inject never sends; way forward: "lyx shuttle interrupt <guid>" sends it behind the pane guard`.
  - A `Text` that holds any control character (`unicode.IsControl`), since reed types `Text` literally with `tmux send-keys -l`, so an ESC byte in it reaches the pane as the Escape key.
    The check is provider-neutral and needs no engine sequence.
    Its message: `shuttle: Inject: input %d's text holds the control character %q, which Inject never types; way forward: send a key as a Key input, and the interrupt key through "lyx shuttle interrupt <guid>"`.
  The checks hold whatever the input's `SettleMS` and whatever inputs surround it.
  The refusal sends nothing and touches no reed method.
- Bound: the `Key` check is against the engine's own sequence and the one alias above, so a tmux spelling of the interrupt key beyond those, such as a raw key code, still passes.
  That is bounded by Inject having no production caller, so every production interrupt key goes through the guarded `Interrupt`.
  An engine whose sequence is empty, such as shuttlefake's default, refuses no `Key`, but still refuses a control character in `Text`.
- The godoc says Inject skips the ready-pane check but refuses the interrupt key, naming `lyx shuttle interrupt`.
- `TestNewRunner_RefusesUnusableToldPaths` and `TestNewDetachedRunner_RefusesUnusableToldPaths` in `run_test.go` pass a lone `{Key: "C-u"}` to `Inject` instead of a lone `{Key: "Escape"}`, so their told-path refusal does not depend on the told check running before the interrupt-key refusal.

Tests in `run_inject_test.go`:

- `TestRunner_Inject_HappyPath_PlaysEveryInputInOrder` plays `{Key: "C-u"}` then the submitted text instead of `Escape` then the text.
  It asserts the `C-u` key, the text and the call log `[Status, SendKey:C-u, SendText:go test ./...]`, which pins that a list with no interrupt key still plays.
- `TestRunner_Inject_DeadStrand_Refuses` and `TestRunner_Inject_UnknownGUID_RefusesBeforeTouchingReed` pass `{Key: "C-u"}` instead of `{Key: "Escape"}`, so they still reach the liveness and `FindRun` refusals they test.
  `fakeEngine.InterruptSequence` is `Escape`, so a lone `Escape` would now stop at the new refusal first.
- New `TestRunner_Inject_RefusesAnyInterruptKey`, a table on a live `withStrand("strand-1")` fixture with `fakeEngine`.
  Its rows are a lone `Escape`; an `Escape` with `SettleMS: 200`; two `Escape`s; `C-u`, then `Escape`, then a submitted text; a lone `{Key: "C-["}`; a lone `{Key: "escape"}`; and a lone `{Text: "\x1b"}`.
  Each row asserts an error containing `lyx shuttle interrupt`, an empty `CallLog` (no `Status`, no key, no text) and no `SendKey` or `SendText` calls.
  The `Text` row's error also contains "control character".
- `cmd/lyx/testdata/test-budget.yaml`: raise `internal/shuttleengine` from 169 to 170.

PATTERN (D7):
`PATTERN.md` sits 20 bytes under `pattern.HeadroomBytes` (15 544 of 15 564), and `patternlint` fails a branch that grows it past that line.
The new entry is therefore paid for by tightening the existing lines below; no rule changes, and the detail each loses already sits in its background file.
After the edit `wc -c PATTERN.md` must be at most 15 564.
The writer measured the set below at 15 560.

Insert this entry in the "Agents and prompts" group, right after `PATTERN-pane-text-injection`:

```markdown
- `PATTERN-guarded-interrupt` — Sending an agent pane the interrupt key: only shuttle's two pane guards; `Runner.Inject` refuses it. — [background](pattern/PATTERN-guarded-interrupt.md)
```

Replace each of these lines whole, in place:

```markdown
- `PATTERN-orch-pane-single-writer` — Typing into the orch session from Go: only the orch watcher, idle-gated but for a hub pause's interrupt; other modules queue an `orchengine` notice.
- `PATTERN-pane-text-injection` — Typing text into an agent pane: only reed's guarded paste-submit. — [background](pattern/PATTERN-pane-text-injection.md)
- `PATTERN-lyx-temp-root` — Creating a temp file or directory: under `internal/lyxtemp`'s run directory, never in `os.TempDir()`. (test) — [background](pattern/PATTERN-lyx-temp-root.md)
- `PATTERN-friction-capture` — Halting a loom run or refusing in webster: Go writes a friction note, archived only once a reflection covers it. (test) — [background](pattern/PATTERN-friction-capture.md)
- `PATTERN-mutation-record` — Adding a mutating fabric verb: it fills a `*Mutations` record, and its result embeds `MutationRecord` under fixed envelope keys. — [background](pattern/PATTERN-mutation-record.md)
- `PATTERN-plan-generation` — Archiving a plan: `_lyx/plan/`'s top-level files hold one generation; a retired one moves to `round-<N>/prior-generation/`. — [background](pattern/PATTERN-plan-generation.md)
- `PATTERN-gate-self-check-parity` — Adding a mechanical gate: its closure and its CLI self-check call one package function and land in one task. — [background](pattern/PATTERN-gate-self-check-parity.md)
- `PATTERN-parent-directive` — Writing a spawned role's top-level stencil: it renders the parent directive; no stencil tells an agent to ask the operator. (test) — [background](pattern/PATTERN-parent-directive.md)
```

`pattern/PATTERN-guarded-interrupt.md` opens with `# PATTERN-guarded-interrupt`, then a one-line statement: an interrupt key reaches an agent pane only through shuttle's two pane guards, and every other path refuses it.
Bullets, one sentence per line:

- The guarded interrupt (`Runner.Interrupt`, `Run.Interrupt`) sends the provider's interrupt sequence only after the ready-agent-pane check and a fresh capture the provider reads as a running turn.
  A pane with no running turn gets nothing and `ErrNoRunningTurn`, and a capture that fails sends nothing.
- A caller that retries an interrupt waits at least the provider's settle time (`Runner.InterruptSettle`) after its first key before it reads again, and sends the retry through the guarded interrupt.
  So lyx's own keys land at least the settle apart and never open the rewind picker.
- The guarded picker close is the one exception: before every interrupt and every verified send it sends exactly one key of the interrupt sequence, only into a capture the provider reads as the rewind picker.
  It then re-captures after the settle, never selects an entry, never sends a second key, and fails the operation with `ErrRewindPickerOpen`.
- `Runner.Inject` skips the ready-pane check, so it refuses any input list holding a key of the engine's interrupt sequence (case-insensitive, `C-[` read as `Escape`) or a `Text` with a control character, whatever its `SettleMS` and neighbours, and names `lyx shuttle interrupt`.
  A tmux spelling of the interrupt key beyond those still passes, bounded by Inject having no production caller.
- The orch watcher's hub-pause interrupt reaches the orch pane through the guarded interrupt and keeps the single-writer exemption (`PATTERN-orch-pane-single-writer`).
- The tests that pin it: `TestRun_Interrupt_PlaysEscape`, `TestGuardedPickerClose` and `TestRunner_Inject_RefusesAnyInterruptKey` in `internal/shuttleengine`.

`pattern/PATTERN-pane-text-injection.md`, last bullet: `Runner.Interrupt` and `Runner.Inject` stay outside this rule as explicit key deliveries, but their interrupt key follows `PATTERN-guarded-interrupt`.
Add that clause, naming the entry, and leave the rest of the bullet as it is.
The file's first bullet already forbids a blind Enter, so the overview line drops that detail without losing it.

**ImpactSummary:** `Runner.Inject` has no production caller, so the refusal changes only its own tests; the PATTERN edits change wording, not rules, and keep the overview under its headroom line.

**Commit:** 5: Refuse the interrupt key in Inject and add PATTERN-guarded-interrupt
