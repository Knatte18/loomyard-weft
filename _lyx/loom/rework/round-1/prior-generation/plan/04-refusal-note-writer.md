# Card 4 — refusal-note-writer

**Create:**
- `internal/websterengine#RefusalNote`
- `internal/websterengine#WriteRefusalNote`
- `internal/websterengine/refusalnote.go`
- `internal/websterengine/refusalnote_test.go`

**Uses:**
- `internal/friction#NotePath`
- `internal/websterengine#writeTriageFrictionNote`

**Intent:**
Add the Go writer for the friction note a `lyx webster` refusal leaves (D3), in `websterengine` beside its triage-note precedent `writeTriageFrictionNote`, so `webstercli` (card 5) and loom's halt scenario test (card 6) write the identical note.

`RefusalNote` carries `Verb` (the subcommand name), `Args` (the invocation's rendered arguments), `Message` (the refusal envelope's `error` text) and `Fields` (`map[string]any`, every other envelope key but `ok` and `error`, which is where a refusal carries its class, such as `plan_drifted` or `batch_failed`).

`WriteRefusalNote` returns nil and writes nothing when `frictionDir` is empty, which is Tier 2 off.
Otherwise it ensures the directory, takes a path from `friction.NotePath(frictionDir, "webster-refusal-"+Verb)`, returns nil when that path is empty (an id `NotePath` rejects), and writes a markdown note: a title line naming `lyx webster <verb>` as refused, the arguments, the refusal text in a fenced block, and one `key: value` line per field in sorted key order.
A write failure is returned wrapped with the path; callers only log it.

`refusalnote_test.go` covers:

- the note's exact content for a `begin-batch` refusal carrying `plan_drifted: true`;
- an empty friction directory writing nothing;
- two refusals of the same verb landing in two notes, through `NotePath`'s free-name suffix;
- a refusal with no extra fields writing no field lines.
