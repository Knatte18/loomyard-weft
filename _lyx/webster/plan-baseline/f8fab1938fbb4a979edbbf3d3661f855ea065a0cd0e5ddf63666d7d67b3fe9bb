# Card 6 — webster-wiring

**Create:**
- `plan:internal/webstercli#TestWireHub_LanguageSkills` -> `func TestWireHub_LanguageSkills(t *testing.T)`
- `plan:internal/webstercli#websterCLI.recoverDeps` -> `func (c *websterCLI) recoverDeps(plan *planparser.Plan, batches []batcher.Batch, st *websterengine.State) websterengine.RecoverDeps`

**Edit:**
- `internal/webstercli#websterCLI`
- `internal/webstercli#websterCLI.wireHub`
- `internal/webstercli#websterCLI.runDeps`
- `internal/webstercli#websterCLI.recoverBatchCmd`
- `internal/webstercli#configLoadError`
- `internal/websterengine/doc.go`
- `internal/webstercli#TestWire_StandaloneMode`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `plan:internal/languageconfig#Load`
- `plan:internal/languageconfig#ErrUnusable`
- `internal/websterengine#RunDeps`
- `internal/websterengine#RecoverDeps`

**Intent:**
`lyx webster`'s wiring loads the `language` module once per process, so `lyx webster run` tells it to the master and `lyx webster recover-batch`, which spawns the recovery strand in its own process, tells it to the recovery strand (D4).
A recovery strand therefore loads the list as it stands when its own process starts, which can be newer than its master's.

`websterCLI` gains the field `languageSkills []string`.
`wireHub` calls `languageconfig.Load(fabricengine.ConfigDirs(loc))` beside its `webster.yaml` load, and on an error returns `configLoadError("language.yaml", err)`; it stores the list in `c.languageSkills`.
`wireStandalone` leaves `c.languageSkills` empty and loads no `language` module, so a standalone run reads no `language.yaml` (D3).
`runDeps` sets `LanguageSkills: c.languageSkills`.
The `websterengine.RecoverDeps` literal in `recoverBatchCmd` moves into the new method `recoverDeps(plan, batches, st)`, which fills the same fields from `c` and the three arguments and sets `LanguageSkills: c.languageSkills`; `recoverBatchCmd` calls it.

`configLoadError` gains a case beside the retired-key one: a cause that is `languageconfig.ErrUnusable` already ends in its own way forward, so the refusal's message is `webster: <file> did not load: <cause>, then re-run the verb`, marked invalid, without the `fix <file> under _lyx/config` clause, which would name the worktree's `_lyx/config` that a hub-wide file never lives in (`PATTERN-hub-wide-config`).
Its doc comment names the new case.
Being invalid, the refusal carries `config_invalid: true` and Master stops the run on it.

What the new load can refuse: every hub-mode `lyx webster` verb now refuses on a hub `language.yaml` that is present but unusable; an absent file resolves the empty list and never refuses (D3).
`contracts/specs/refusal-spec.md`'s webster row "config file content invalid" adds `language.yaml` to the files whose prologue load it names, its trigger naming a `skills` value that is not a list of strings, an entry that is empty or carries surrounding whitespace, or a worktree local override file that holds `skills`, which the Guard classification refuses, and its way forward adds, quoting the message: for `language.yaml`, fix it with "lyx config language" from the prime, then re-run the verb.
The webster row "config file unreadable" names `language.yaml` in its trigger beside `batcher.yaml` and `webster.yaml`; its class (transient) and way forward (re-run the verb) stay as they are.
The "A broken config stops Master" section of `internal/websterengine/doc.go` names `language.yaml` beside `webster.yaml` and `batcher.yaml`: a present file with unusable content refuses with `config_invalid: true` and stops Master, an unreadable one refuses transient.

`TestWireHub_LanguageSkills` (in `internal/webstercli/wiring_test.go`, over `hubLocation`, writing `<BoardDir>/_lyx/config/language.yaml` under `fabricengine.BoardDir(hub)`) is table-driven.
Writing that file creates the board dir, so every other module `wireHub` loads resolves from an absent board default file: `shuttle`, `reed`, `webster` and `batcher` degrade to their templates, `loom` only warns, and `modelspec.LoadRegistry` with no layer file returns its builtins, so `wire` reaches the `language` load in every row.
- `skills` listing `scribe:golang-comments`, `scribe:golang-testing`: `wire` in `preflight.ModeHub` succeeds, and `c.languageSkills`, `c.runDeps().LanguageSkills` and `c.recoverDeps(nil, nil, nil).LanguageSkills` all equal that list;
- `skills` holding an entry with a leading space: `wire` fails with a `configLoadRefusal` whose `invalid` is true and whose message contains `language.yaml`, `skills` and `lyx config language` and does not contain `under _lyx/config`, which reaches the extended refusal row and the new `configLoadError` case.
The absent-file degrade is card 1's `TestLoad_ResolvesSkills` row, and `TestWire_ModeHubSelectsHubMode` already runs hub wiring with no `language.yaml`.
`TestWire_StandaloneMode` first writes the target's `_lyx/config/language.yaml` listing `scribe:a`, then additionally asserts `c.languageSkills` is empty, so a standalone wiring that read a worktree file would fail it.

**ImpactSummary:** `wireHub` gains one hub-file load and the CLI one field, read by `runDeps` and the new `recoverDeps` that replaces `recover-batch`'s inline literal; `configLoadError` gains one case, and `begin-batch` and every other hub verb only gain the load's refusal.
