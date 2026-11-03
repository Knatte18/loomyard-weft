# Config --set replaces a whole open map from a flow mapping

From [issue #19](https://github.com/Knatte18/loomyard/issues/19).

`lym config drift --set 'capture={command: "...", dir: "...", timeout_min: 30}'` was refused with `unknown config key(s): capture, compare`, although the key description in drift's template prescribes that whole-map form.
Only the per-key form `capture.timeout_min=45` worked.

## Cause

`capture` and `compare` are open maps with a declared shape.
yamlengine's `SetValues` and `SetSparse` knew a key `P.<name>` under an open map `P`, and `P` itself only when the template holds a scalar or a list there.
A mapping-valued `P`, such as drift's `capture: {}`, was never a known key itself.

## Change

- `internal/yamlengine`: every declared open map `P` is now a known key and is listed in `Known`.
  A pair at a mapping-valued `P` replaces it whole with the YAML flow mapping its value spells, `{}` included, on both the scaffolded and the sparse write path (`setWholeOpenMap`).
  A value that is not one flow mapping is refused, and the refusal names `P` and the per-key form `P.<key>`.
  Under a declared shape, every key inside the mapping is checked against the shape (`mappingShapeError`), so `capture={command: ..., timeoutt: 1}` is refused with `capture holds no key timeoutt`, the same wording the per-key form gives.
  A single-value field takes whatever it holds.
  Scalar-or-list open maps keep their behavior.
- Docs: the yamlengine package doc, `SetValues`/`SetSparse` doc comments, configengine's and configcli's package docs and drift's `ConfigSpec` comment describe the whole-map form.
  drift's template description already prescribed this form, so it stays as it is.

## Design decisions

- The whole-map form applies to every module's mapping-valued open maps, not only drift's, so `lym config gate --set 'tiers={...}'` also replaces the tier map whole.
  This is the generic reading of "a mapping whose keys the user owns", and the key already accepted `P.<name>` flow mappings for entries.
- The `lym config` cobra `Long` help is left unchanged: any added line exceeds its 2000-byte cap (`TestCLIForm_EveryCommand`), and the configcli package doc carries the new form with a drift example.

## Tests

- `TestSetSparse` and `TestSetValues` gain rows for a whole open map replaced or emptied, a whole shaped map written with its scalar tags, and refusals for a key the shape lacks and a value that is no flow mapping; `TestSetValues_UnknownKeys` pins `P` in `Known`.
- `TestConfigSpec_SetWholeSteps` in driftengine sets `capture` and `compare` whole through `configengine.Set`, loads them back as declared steps, and pins that a misspelled key inside a whole step is refused with the file left unchanged.
