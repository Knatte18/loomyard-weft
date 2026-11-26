# Card 2 — config-moved-keys

**Create:**
- `plan:internal/configengine#MovedKeyIn` -> `func MovedKeyIn(spec Spec, doc []byte) (oldKey, newPath string, found bool, err error)`

**Edit:**
- `internal/configengine#Spec`
- `internal/configengine#ValidateLocal`
- `internal/configengine#load`
- `internal/configengine#Set`
- `internal/configengine#Unset`
- `internal/configengine_test#TestValidateLocal`
- `internal/configengine_test#TestLoad_LayerStack`
- `internal/configengine_test#TestSet`
- `internal/configengine/doc.go`
- `PATTERN.md`
- `pattern/PATTERN-config-layers.md`

**Uses:**
- `plan:internal/configengine#KeyName`
- `internal/configengine#nodeAt`
- `internal/configengine#boardDirWayForwardClause`

**Intent:** Give `configengine.Spec` a move table and refuse an old name after the move (D4), in the layers configengine owns.

`Spec` gains `MovedKeys map[string]string`: each old dotted key the module moved, mapped to its new dotted path.
It stays out of `Categories`, `OpenMaps` and `RetiredKeys`.

`MovedKeyIn(spec, doc)` returns the first old key, in sorted order, that `doc` holds under D4's matching rule:
- it matches a unit whose path equals the old key exactly, never a path below it;
- an old key equal to its new path's first segment, such as `discussion` moving to `discussion.model`, matches only a non-mapping value, so a `discussion:` mapping is the new area and never the old key;
- the old key's category does not matter.
Every caller passes a document its own read already parsed, so a parse failure here is a fail-loud wiring guard in `refusal-spec.md`'s sense: it is returned unmarked and takes no reaching test.

The refusal, in every site below, names the old and new full names through `KeyName` and the site's way forward:
`key <module>.<old> was moved to <module>.<new>; way forward: <site's way forward>`.
- `ValidateLocal` runs the check before its per-path checks, so in a local file it replaces the "not in the template" refusal for the table's old names.
  Its way forward is `run "lym config reconcile --apply" in this worktree`, which moves the key (card 6); the local file is untracked, so nothing needs committing.
- `load` runs the check over the default and worktree layers it read, before `warnUnknownKeys`.
  A default-layer hit names `run "lym config reconcile --layer default --apply" from the hub's prime worktree` and ends with `boardDirWayForwardClause`, as every refusal naming a verb that writes under the board dir does, so an agent session that receives it reports FAILED and the orch runs the command;
  a worktree-layer hit names `run "lym config reconcile --apply" in this worktree, then commit the moved _lym/config/<module>.yaml`, since the reconcile writes the tracked file without committing it.
  Both are marked `ErrInvalid` and name the file.
- `Set` refuses a pair whose key is an old name, before anything is written, naming the command for the layer it was told with the new path:
  `run "lym config <module> --layer <layer> --set <new>=<value>"`, the pair's value as given, so repeating it writes the same layer.
- `Unset` admits an old name and removes it as any other key, so the operator's removal route stays open; its doc comment says so.
Bound: the check refuses only names the table holds, and admits nothing the template and categories would refuse; a spec with no table is judged as today.

Reaching tests, tier0, each a row of an existing table on a test spec whose template holds `area: {key: 0, model: ""}` and that sets `MovedKeys: {"old_key": "area.key", "area": "area.model"}`:
- `TestValidateLocal` (specs built inline per row): a local document `old_key: 2` is refused naming `<module>.old_key`, `<module>.area.key` and `lym config reconcile --apply`; `area: x` (scalar) is refused as moved; `area: {key: 1}` is admitted, returning nil.
- `TestLoad_LayerStack`, whose `layerFixture` stacks a default and a worktree file: a default-layer file holding `old_key` is refused naming `lym config reconcile --layer default --apply` and the board-dir clause; a worktree-layer file holding it is refused naming `lym config reconcile --apply` and the commit. The table gains a `movedKeys` field its spec takes.
- `TestSet`, whose table gains a `movedKeys` field: a worktree set of `old_key=3` is refused naming `--layer worktree --set area.key=3` and writes nothing; a local-layer set of it names `--layer local`; an unset of `old_key` from a file holding it removes it.

`doc.go`'s "The local rule" section gains the move table: what it maps, that each layer refuses an old name with its reconcile command, and that `Unset` removes one.

`PATTERN.md` replaces the `PATTERN-config-layers` line with the declared `changes` text, as this whole line:

```markdown
- `PATTERN-config-layers` — Loading, writing or changing config: layers only through `configengine`; a hub base file holds only tuned units; a meaning change ships a reinterpretation, a key move a move table, both run by `hubreconcile`; a per-task beam setting is read only via `dobbyengine`'s one lookup. — [background](pattern/PATTERN-config-layers.md)
```

`pattern/PATTERN-config-layers.md` takes the new rule text in its opening line, and its "Keys the template lacks" section gains the move table, per the record's note to the plan writer:
a key its module's move table names is moved by every reconcile, refused at load with that layer's reconcile command, `lym config reconcile --apply` or `lym config reconcile --layer default --apply`, as the way forward, and removable by `--unset`.
The "Layers" rule on line 7, that no step removes an unknown base-layer key but the operator's `lym config reconcile` or `--unset`, gains an exception:
a key the module's move table names is moved, or dropped when null or when its new path is set, by every reconcile, the hub walk included.
The sentence at line 73, that only the operator removes a key the running template lacks from a hub base or worktree file, and the sentence at lines 77-78, that every other reconcile keeps such a key, the binary-change walk included, gain the same exception.
Its "Config reinterpretation" rule adds that a key move ships a move table on the module's `ConfigSpec`, which `configsync` applies on every reconcile and `hubreconcile`'s walk carries to the board's `run_settings`.

**ImpactSummary:** Adds one Spec field and one check reached from configengine's local validation, strict load and set; no module fills the table yet, so no current load changes, and the PATTERN line and background change in prose only.
