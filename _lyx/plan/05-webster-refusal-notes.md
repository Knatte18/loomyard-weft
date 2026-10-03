# Card 5 — webster-refusal-notes

**Edit:**
- `internal/webstercli/cli.go`

**Create:**
- `internal/webstercli/refusalnote.go`
- `internal/webstercli/refusalnote_test.go`

**Uses:**
- `plan:internal/websterengine#WriteRefusalNote`
- `plan:internal/websterengine#RefusalNote`
- `internal/output#ErrFields`

**Intent:**
Every refusal envelope a `lyx webster` verb prints, except `validate`'s, writes one friction note (D3), at the one place every verb's output passes through, so no agent has to remember to record it.

`refusalnote.go` adds `(c *websterCLI) noteRefusals(cmd *cobra.Command) *cobra.Command`, which wraps `cmd.RunE`.
The wrapper points the command's output at a tee into a buffer for the duration of the original `RunE`, restores it afterwards, and decodes the last non-empty line written as the JSON envelope.
When `c.frictionDir` is non-empty and the envelope's `ok` is false, it calls `websterengine.WriteRefusalNote` with the command name, the arguments (positional args, then every flag the invocation set, rendered `--name=value` through `cmd.Flags().Visit`), the envelope's `error`, and its remaining keys.
A note-write failure, or an output it cannot decode, only logs a warning: the verb's exit code and envelope stay exactly what the original `RunE` produced.
An `ok: true` envelope, such as `begin-batch`'s `paused: true`, writes nothing.

`Command` registers its subcommands through a new `(c *websterCLI) addVerbs(parent *cobra.Command)`, which wraps every verb through `noteRefusals` except `validate`: `validate` is a lint an agent iterates against, so its findings are expected rather than friction.
Refusals raised in `resolvePersistentPreRun` stay unnoted, since they fire before the friction directory is resolved.

What this lets through, and the bound: one local note file per refusal and nothing filed by itself; Tier 2 off (an empty friction directory) writes nothing, and the reflection's content rule (card 10) bounds what an issue may quote from the arguments and refusal text.

`refusalnote_test.go` (untagged, no hub needed) drives commands built by `addVerbs` on a bare parent over a hand-built `websterCLI` with a temporary friction directory:

- `begin-batch x`, `recover-batch x` and `rebaseline --card abc` each refuse on argument parsing and write one note holding the verb, the arguments and the refusal text, with the envelope byte-identical to the unwrapped command's;
- a stub verb printing an `output.ErrFields` refusal with `plan_drifted: true` writes a note carrying that field;
- a failing `validate` (missing plan directory) writes no note;
- an empty friction directory writes no note and leaves the envelope unchanged;
- an unwritable friction directory leaves the exit code and envelope unchanged.

**ImpactSummary:** one registration change in `webstercli.Command`; every non-`validate` verb gains a pass-through output wrapper whose only side effect is an optional note file.
