# Card 9 — friction-capture-docs

**Prosa:**
- `CONSTRAINTS.md`
- `docs/overview.md`
- `internal/friction/doc.go`

**Intent:**
Record the cross-cutting rules this task establishes (D7) and bring the module docs in line with the cards before it.

`CONSTRAINTS.md` gains a `## Friction Capture Invariant` section after the Friction Leaf Invariant:

- with Tier 2 on, a loom halt (`blocked` or `failed`, under `run` or `step`) and every `lyx webster` refusal but `validate`'s write one Go-authored friction note, through `friction.NotePath`, from `loomcli` and webster code respectively, and a note-write failure never changes the verb's exit or envelope;
- no code path archives or deletes a friction note before a reflection has covered it: `frictionengine.Reflect` archives only the notes its covered-notes record names, and only after a clean return or on a finished report;
- the reflection files through `lyx selfreport create` only;
- what enforces each clause: the `loomcli`, `webstercli`, `websterengine` and `frictionengine` tests the earlier cards add, and review for the rest.

`docs/overview.md`:

- the `internal/frictionengine/` tree line: the reflection runs in the terminal Friction-Reflect row and after a `blocked` or `failed` halt, under `run` and `step`, and is re-entrant across a killed driving process;
- the `awaiting` line keeps "triggers no friction reflection" and adds that a `blocked` or `failed` halt writes a Go halt note and reflects;
- the friction-notes line names the Go-written halt and webster refusal notes beside the agents' notes, and the driver's repair records written into the friction directory;
- the generic `step` envelope line adds the `friction` key on the success and every error envelope.

`internal/friction`'s package doc adds that, besides the agent-written notes the directive asks for, `loomcli` and webster code write Go-authored halt and refusal notes through `NotePath`, keeping this package free of feature imports.
