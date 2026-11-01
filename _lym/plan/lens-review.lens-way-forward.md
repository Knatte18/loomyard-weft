```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward. Each refusal, stop and recovery step the cards add was checked for the next step it names and whether that step can work from the state it leaves.

**Findings**

1. MEDIUM — Card 8, step 3 (D6/D7, `{strand}` entry with no `@lym_strand_name`): the next step it names cannot work.
   - The message says "&lt;label&gt; needs an agent pane; select one and click again".
   - Card 6 draws a `window: agents` entry only in the agent row, which shows only while the active pane carries `@strand`. So this branch fires only when the operator is already on an agent pane whose `@lym_strand_name` is missing.
   - That happens to a strand an older lym launched (card 7's Intent: "a strand spawned by an older lym gets it at the next attach").
   - Selecting another agent pane or clicking again changes nothing. The option comes back only through the attach pre-flight's `reassertStrandPaneOptionsLocked` (card 7).
   - Evidence: `_lym/plan/08-reed-button-verb.md` step 3, `_lym/plan/07-strand-name-pane-option.md` Intent, `_lym/plan/06-render-button-row.md` (agent row), `internal/reedengine#Engine.reassertStrandPaneOptionsLocked`.
   - Fix: the message should name the step that sets the option, reattaching the session (`lym reed attach`).
   - `TestRunButtonVia`'s agent-pane row should assert that wording.

2. LOW — Card 3 (D2, invalid-entry Warn): the Warn cannot name the file that holds the bad entry.
   - `placeButtons(e.cfg.Buttons, place, "reed.yaml")` gets the node only after the layers are stacked (card 6), so `source` is always the label `reed.yaml`.
   - The bad entry can sit in the hub base or in the worktree file, and nothing tells the operator which one to fix.
   - D2 asks for "one Warn naming the file, group and index".
   - Evidence: `_lym/plan/03-button-entries.md` ("`source` is the file label the render passes, `reed.yaml` or `desktop.machine.yaml`"), `_lym/plan/06-render-button-row.md` (the call with `"reed.yaml"`), `internal/configengine/layers.go:stackLayers` (the merge drops which layer a value came from).
   - Fix: name both candidate files in the Warn, or find the layer through `configengine.Layers`/`Provenance` before warning.

3. LOW — Card 6 (D4, missing beam anchor): the Warn names no next step.
   - When a task's parent beam has no anchor here, the `{beam}` entries are left out with "one Warn per entry naming its label and the missing beam's `Name`".
   - The Warn does not say how to get the button back: bring the beam pair onto this machine (`lym adopt &lt;beam&gt;`), after which the row re-renders at the next attach.
   - Evidence: `_lym/plan/06-render-button-row.md` composeButtonRowLocked bullets; `internal/beamcli/target.go:absentBeamRefusal`, which names `lym adopt` for the same absent-pair state.

4. LOW — Card 8, step 5 (unresolvable executable while raising the confirm prompt): a click does nothing and the operator sees nothing.
   - The failure is logged at Warn and nothing runs. No `display-message` is shown, though every other stop in `runButtonVia` shows one.
   - The operator gets no prompt and no message, and nothing points them to the log.
   - Evidence: `_lym/plan/08-reed-button-verb.md` step 5 ("An unresolvable executable is logged at Warn and runs nothing").
   - Fix: show the same status message the other stops use, naming the label and the log.

5. LOW — Card 8, step 3, confirmed path (strand name fails `agentname.Parse`): the message names no next step.
   - "the strand name &lt;name&gt; is not a strand name" is shown, but not what to do, such as clicking the button again from the agent pane.
   - Only a direct or forged call reaches this branch, since the pinned confirmed line carries the name it read.
   - Evidence: `_lym/plan/08-reed-button-verb.md` step 3.

6. LOW — Card 8, steps 1 and 2 (empty or unreadable spec): the stop shows nothing in the bar.
   - An empty or undecodable `@lym_button_spec` is logged at Warn and runs nothing, with no status message.
   - A session pinned by an older binary can get here: card 9's Bound says the bindings can be newer than the session's options. An `lymbtn` range can then exist without a spec this verb decodes.
   - The operator gets no next step, such as reattaching to re-render the row.
   - Evidence: `_lym/plan/08-reed-button-verb.md` step 1, `_lym/plan/09-button-click-binding.md` Bound.

**Refusals that do lead somewhere**

These ones name a next step that works from the state they leave: the changed-spec stop, the hash mismatch, the gone run directory, a failed run, a failed `display-message`, the shape refusal of a machine-layer mapping group, a `reed.yaml` `buttons: null`, the local-layer refusals, the closure and origin read failures, and `validateButtonFlags`.
- After a changed spec or a hash mismatch, clicking again works, because the row and the spec render together.
- After a gone run directory, the next attach re-renders the row: card 4 falls back to `PrimeAnchor`.
- A failed run shows its first output line, which carries `ResolveBattenPrime`'s worktree.
- The machine-layer shape refusal from `ValidateMachine` names "remove it from the machine override".
- The `buttons: null` fill refusal names the config file, as `internal/configengine/config.go:234-236` shows.

**Tool calls**
- Grepped `internal/configengine` and `internal/yamlengine` for `FillMissing`, `MarkInvalid` and `shapeMismatch`: my context did not show how the template-fill refusal is worded or wrapped.
- Read `internal/configengine/config.go:228-246`: to confirm the fill refusal is marked ErrInvalid and names a file.
