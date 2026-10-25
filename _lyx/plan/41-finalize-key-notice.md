# Card 41 — finalize-key-notice

**Edit:**
- `internal/fabricengine#ConfigChanges`
- `internal/fabricengine#ReadConfigChanges`
- `internal/landingshed#Finalize.configChangeNotice`
- `internal/landingshed#TestFinalize_ConfigChangeNotice`
- `internal/fabricengine_test#TestReadConfigChanges`
- `internal/fabricengine/doc.go`

**Create:**
- `internal/configengine#KeyChange`
- `internal/configengine#ChangedKeys`
- `internal/configengine_test#TestChangedKeys`

**Intent:**
Report the per-worktree config notice at Finalize key by key (D11).
`ConfigChanges` gains `Contents`, per changed file the bytes at the fork base, the task tip and the parent tip (nil for a file absent at a revision); `ReadConfigChanges` reads them with `git show <rev>:<path>` in the records repository for the files it already reports, so a file whose text changed is still a candidate.
`configengine.ChangedKeys` parses the three documents as YAML, flattens each to dotted key paths with their values, and returns, sorted by key, each key whose tip value differs from its base value and from its parent value, as `KeyChange{Key, Base, Tip, Parent}` with an absent key or file reading `absent`; a document that does not parse is an error naming it.
`configChangeNotice` names, per file and its module, each changed key with its base, tip and parent values (`loom.code_max_bounces: base 2, tip 1, parent 2`); it is silent when the tip changes no key, as for an empty file or comments only, and when the parent already holds the tip's value; a read or parse failure is still logged and reported in the notice instead of stopping the landing.
`fabricengine/doc.go`'s `ReadConfigChanges` paragraph names the contents.
Tests: `TestChangedKeys` (tier0, parallel) rows: a comments-only change returns none; a key the parent already holds returns none; a changed nested key returns its dotted path with three values; a key absent at the base reads `absent`; an unparseable tip is an error. `TestReadConfigChanges` (tier1) gains rows: a reported file carries its bytes at the base, the tip and the parent tip, and a file the task added reads nil at the base.
`TestFinalize_ConfigChangeNotice` rows: a comments-only change is silent, a key change names the key with base, tip and parent values, `absent` included.

**ImpactSummary:** The config-change record carries file contents and Finalize's one notice composes keys from them; its single wiring in loomcli passes the record through unchanged.
